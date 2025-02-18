pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE', description: 'Full Docker image path to update in Helm chart')
        string(name: 'BUILD_STATUS', description: 'Status of the upstream build')
    }

    environment {
        HELM_CHART_REPO = 'https://github.com/samueldison/springboot-java-app.git'
        CHART_PATH = 'values.yaml'
    }

    stages {
        stage('Clone Helm Chart Repo') { 
            steps {
                git branch: 'main', url: "${HELM_CHART_REPO}"
            }
        }

        stage('Update Helm Chart with SED') {
            steps {
                script {
                    def imageParts = params.DOCKER_IMAGE.tokenize('/')
                    if (imageParts.size() < 2) {
                        error("Invalid DOCKER_IMAGE format: Expected 'registry/image:tag', but got '${params.DOCKER_IMAGE}'")
                    }

                    def registry = imageParts[0]
                    def imageNameTag = imageParts[1]

                    def tagParts = imageNameTag.tokenize(':')
                    if (tagParts.size() < 2) {
                        error("Invalid image format: Expected 'image:tag', but got '${imageNameTag}'")
                    }

                    def imageName = tagParts[0]
                    def tag = tagParts[1]

                    echo "Registry: ${registry}, Image: ${imageName}, Tag: ${tag}"

                    sh """
                        sed -i 's|repository: .*|repository: ${registry}/${imageName}|g' ${CHART_PATH}
                        sed -i 's|tag: .*|tag: ${tag}|g' ${CHART_PATH}
                    """

                    sh "grep -E 'repository:|tag:' ${CHART_PATH}"
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh '''
                            git config user.email "orionhouse83@gmail.com"
                            git config user.name "Orion House"
                            git remote set-url origin https://${GIT_USER}:${GIT_PASS}@github.com/samueldison/springboot-java-app.git

                            if [[ -n $(git status -s) ]]; then
                                git add ${CHART_PATH}
                                git commit -m "Update image to ${DOCKER_IMAGE}"
                                git push origin main
                            else
                                echo "No changes to commit"
                            fi
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            emailext body: """\
                Helm Chart Successfully Updated!

                Docker Image: ${params.DOCKER_IMAGE}
                Build Status: ${params.BUILD_STATUS}

                Helm Chart Repository has been updated with the latest image.

                Changes applied to: ${CHART_PATH}
            """,
            subject: "Helm Chart Update - Successful",
            to: 'samuelhaddison71@gmail.com'
        }

        failure {
            emailext body: """\
                Helm Chart Update Failed!

                Docker Image: ${params.DOCKER_IMAGE}
                Build Status: ${params.BUILD_STATUS}

                Please investigate the Helm Chart update process.
            """,
            subject: "Helm Chart Update - Failed",
            to: 'samuelhaddison71@gmail.com'
        }
    }
}
