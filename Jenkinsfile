pipeline {
    agent any
    environment {
        REGISTRY = "duocpv1101/devopslab"
        // Tag images
        // OLDTAG = sh(script: "cat manifest/deployment.yaml |grep image |awk '{print \$2}'|cut -d ':' -f 2", returnStdout: true).trim()
        // NEWTAG = "${OLDTAG.toInteger() + 1}"
        // OLDIMG = sh(script: "cat manifest/deployment.yaml |grep image |awk '{print \$2}'|awk -F'/' '{print \$2}'", returnStdout: true).trim()
        // REPIMG = sh(script: "echo ${OLDIMG}|awk -F':' '{print \$1}'", returnStdout: true).trim()
        // NEWIMG = "${REPIMG}:${NEWTAG}"
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
                sh 'ls -al app'
                sh 'ls -al manifest'
            }
        }
        stage('Build and Push Docker Image') {
            environment {
                    OLDTAG=sh(script: "cat manifest/deployment.yaml |grep image |awk '{print \$2}'|cut -d ':' -f 2", returnStdout: true).trim()
                    NEWTAG="${OLDTAG.toInteger() + 1}"
                }
            steps {
                script {
                    dockerImage = docker.build("${REGISTRY}:${NEWTAG}", "-f app/Dockerfile app")
                    withDockerRegistry(credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/') {
                    dockerImage.push()
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                    OLDTAG=sh(script: "cat manifest/deployment.yaml |grep image |awk '{print \$2}'|cut -d ':' -f 2", returnStdout: true).trim()
                    NEWTAG="${OLDTAG.toInteger() + 1}"
                    OLDIMG=sh(script: "cat manifest/deployment.yaml |grep image |awk '{print \$2}'|awk -F'/' '{print \$2}'", returnStdout: true).trim()
                    REPIMG=sh(script: "echo ${OLDIMG}|awk -F':' '{print \$1}'", returnStdout: true).trim()
                    NEWIMG="${REPIMG}:${NEWTAG}"
                }
            steps {
                withCredentials([string(credentialsId: 'gittoken', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        cd manifest
                        git config user.email "duocpv1101@gmail.com"
                        git config user.name "Duocpv0101"
                        sed -i "s/${OLDIMG}/${NEWIMG}/g" deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update deployment image to version ${NEWTAG}"
                        git push -f https://${GITHUB_TOKEN}@github.com/Duocpv0101/nginx-argocd-manifest.git HEAD:main
                    '''
                }
            }
        }
        stage('Remove Unused docker image') {
            environment {
                    OLDTAG=sh(script: "cat manifest/deployment.yaml |grep image |awk '{print \$2}'|cut -d ':' -f 2", returnStdout: true).trim()
                    NEWTAG="${OLDTAG.toInteger() + 1}"
                }
            steps{
                sh "docker rmi ${REGISTRY}:${NEWTAG}"
            }
        }
    }
}