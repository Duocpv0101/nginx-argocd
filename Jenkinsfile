pipeline {
    agent any
    environment {
        REGISTRY = "duocpv1101/devopslab"
    }
  
    stages {
        stage('Checkout') {
            steps {
                sh 'mkdir -p app manifest'
                dir('app'){
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Duocpv0101/nginx-argocd.git'
                }
                dir('manifest'){
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Duocpv0101/nginx-argocd-manifest.git'
                }
            }
        }
        stage('Set ENV') {
            steps {
                script {
                    def OLDTAG = sh(script: "cat manifest/app/deployment.yaml |grep image |awk '{print \$2}'|cut -d ':' -f 2", returnStdout: true).trim()
                    def NEWTAG = "${OLDTAG.toInteger() + 1}"
                    def OLDIMG = sh(script: "cat manifest/app/deployment.yaml |grep image |awk '{print \$2}'|awk -F'/' '{print \$2}'", returnStdout: true).trim()
                    def REPIMG = sh(script: "echo ${OLDIMG}|awk -F':' '{print \$1}'", returnStdout: true).trim()
                    def NEWIMG = "${REPIMG}:${NEWTAG}"
                    env.NEWIMG = NEWIMG
                    env.OLDIMG = OLDIMG
                    env.NEWTAG = NEWTAG
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${REGISTRY}:${NEWTAG}", "-f app/Dockerfile app")
                    withDockerRegistry(credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/') {
                    dockerImage.push()
                    sh "docker rmi ${REGISTRY}:${NEWTAG}"
                    }
                }
            }
        }
        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'gittoken', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        cd manifest
                        git config user.email "duocpv1101@gmail.com"
                        git config user.name "Duocpv0101"
                        sed -i "s/${OLDIMG}/${NEWIMG}/g" app/deployment.yaml
                        git add app/deployment.yaml
                        git commit -m "Update deployment image to version ${NEWTAG}"
                        git push -f https://${GITHUB_TOKEN}@github.com/Duocpv0101/nginx-argocd-manifest.git HEAD:main
                    '''
                }
            }
        }
    }
}