pipeline {
    agent any
    environment {
        // 镜像仓库配置
        DOCKER_HUB = "sitanduat.azurecr.io"
        SERVICE_WAIT_TIME = "30"

        // element-web 项目固定参数（与deployment yaml对齐）
        PROJECT_NAME = "element-web"
        DEPLOYMENT_NAME = "element-web"
        K8S_NAMESPACE = "sit-im"
        K8S_YAML_FILE = "element-web.yaml"
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: env.BRANCH_NAME ?: '', description: '构建分支')
        choice(
            name: 'DEPLOY_TARGET',
            choices: ['sit'],
            description: '选择部署环境'
        )
    }

    stages {
        stage('初始化配置（环境+版本）') {
            steps {
                script {
                    // 分支简单合法性校验
                    if (!params.BRANCH_NAME?.trim()) {
                        error("❌ BRANCH_NAME 分支参数不能为空！")
                    }
                    env.RAW_BRANCH_NAME = params.BRANCH_NAME.trim()
                    def timestamp = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
                    env.DOCKER_IMAGE_TAG = "${env.RAW_BRANCH_NAME}-${timestamp}"
                    env.FULL_IMAGE_NAME = "${DOCKER_HUB}/${DEPLOYMENT_NAME}:${env.DOCKER_IMAGE_TAG}"

                    echo "====================================="
                    echo "原始分支：${env.RAW_BRANCH_NAME}"
                    echo "镜像TAG：${env.DOCKER_IMAGE_TAG}"
                    echo "完整镜像地址：${env.FULL_IMAGE_NAME}"
                    echo "部署K8s命名空间：${K8S_NAMESPACE}"
                    echo "====================================="
                }
            }
        }

        stage('拉取Git代码') {
            steps {
                echo "===== 拉取 ${params.BRANCH_NAME} 分支代码 ====="
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "refs/heads/${params.BRANCH_NAME}"]],
                    userRemoteConfigs: [[
                        url: 'git@github.com:remi-team/element-web.git',
                        credentialsId: 'ssh-key-git'
                    ]],
                    extensions: [
                        [$class: 'CleanBeforeCheckout'],
                        [$class: 'PruneStaleBranch']
                    ]
                ])

                // monorepo pnpm项目校验
                sh """
                    ls -l pnpm-lock.yaml || { echo '❌ 未找到 pnpm-lock.yaml，代码拉取异常'; exit 1; }
                    ls -l pnpm-workspace.yaml || { echo '❌ monorepo 配置 pnpm-workspace.yaml 缺失'; exit 1; }
                    ls -l apps/web/Dockerfile || { echo '❌ 未找到 apps/web/Dockerfile'; exit 1; }
                    ls -l ${K8S_YAML_FILE} || { echo "❌ 未找到K8s部署模板 ${K8S_YAML_FILE}"; exit 1; }
                """
            }
        }

        stage('构建并推送Docker镜像') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_REGISTRY_USER',
                        passwordVariable: 'DOCKER_REGISTRY_PASS'
                    )]) {
                        sh """
                            echo "🔐 登录镜像仓库 ${DOCKER_HUB}"
                            docker login ${DOCKER_HUB} -u ${DOCKER_REGISTRY_USER} -p ${DOCKER_REGISTRY_PASS}
                        """
                    }

                    echo "🐳 使用BuildKit构建镜像，上下文=仓库根目录，Dockerfile=apps/web/Dockerfile"
                    sh """
                        export DOCKER_BUILDKIT=1
                        docker build \
                            -f apps/web/Dockerfile \
                            -t ${FULL_IMAGE_NAME} \
                            --build-arg USE_CUSTOM_SDKS=false \
                            --build-arg JS_SDK_REPO="https://github.com/matrix-org/matrix-js-sdk.git" \
                            --build-arg JS_SDK_BRANCH="master" \
                            .
                    """

                    echo "📤 推送镜像至仓库"
                    sh "docker push ${FULL_IMAGE_NAME}"

                    // 清理本地镜像
                    sh "docker rmi ${FULL_IMAGE_NAME} || true"
                    echo "✅ 镜像构建推送完成"
                }
            }
        }

        stage('部署到Kubernetes集群') {
            steps {
                script {
                    def yamlTemplatePath = "${WORKSPACE}/${K8S_YAML_FILE}"
                    // 替换镜像占位符 {{FULL_IMAGE_NAME}}
                    sh """
                        cp ${yamlTemplatePath} ${yamlTemplatePath}.bak
                        sed -i "s#{{FULL_IMAGE_NAME}}#${FULL_IMAGE_NAME}#g" ${yamlTemplatePath}
                        echo "📌 替换后Deployment镜像配置："
                        grep "image:" ${yamlTemplatePath}
                    """

                    withCredentials([file(credentialsId: 'kubeconfig-sit', variable: 'KUBECONFIG_FILE')]) {
                        sh """
                            export KUBECONFIG=\${KUBECONFIG_FILE}
                            echo "🚀 kubectl apply 应用资源"
                            kubectl apply -f ${yamlTemplatePath} -n ${K8S_NAMESPACE}

                            echo "⏳ 等待滚动更新就绪（超时5分钟）"
                            kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} --timeout=300s

                            echo "📊 查看运行Pod"
                            kubectl get pods -l app=${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}
                        """
                    }
                    echo "✅ element-web 部署到K8s sit集群成功！"
                }
            }
        }
    }

    post {
        success {
            echo "=================================================="
            echo "🎉 element-web 部署成功"
            echo "📌 部署镜像：${FULL_IMAGE_NAME}"
            echo "📌 K8s命名空间：${K8S_NAMESPACE}"
            echo "=================================================="
        }
        failure {
            echo "=================================================="
            echo "❌ element-web CI/CD流水线失败！排查方向："
            echo "1. 代码拉取：分支名称、git密钥权限"
            echo "2. Docker构建：确认docker支持BuildKit，检查apps/web/Dockerfile语法"
            echo "3. 镜像推送：镜像仓库账号权限"
            echo "4. K8s部署：kubeconfig权限、configmap element-web-config是否存在"
            echo "=================================================="
        }
        always {
            cleanWs(
                deleteDirs: true,
                notFailBuild: true
            )
            echo "✅ 工作空间清理完成"
        }
    }
}
