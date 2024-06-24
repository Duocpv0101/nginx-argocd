pipeline {
    agent any
    environment {
        REGISTRY = "duocpv1101/devopslab"
        // Tag images
        OLDTAG = sh(script: "cat app-manifest/deployment.yaml |grep image |awk '{print \$2}'|cut -d ':' -f 2", returnStdout: true).trim()
        NEWTAG = "${OLDTAG.toInteger() + 1}"
        OLDIMG = sh(script: "cat app-manifest/deployment.yaml |grep image |awk '{print \$2}'|awk -F'/' '{print \$2}'", returnStdout: true).trim()
        REPIMG = sh(script: "echo ${OLDIMG}|awk -F':' '{print \$1}'", returnStdout: true).trim()
        NEWIMG = "${REPIMG}:${NEWTAG}"
    }
  
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Duocpv0101/nginx-argocd.git'
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${REGISTRY}:${NEWTAG}")
                    withDockerRegistry(credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/') {
                    dockerImage.push()
                    }
                }
            }
        }
        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'gittoken', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "duocpv1101@gmail.com"
                        git config user.name "Duocpv0101"
                        sed -i "s/${OLDIMG}/${NEWIMG}/g" app-manifest/deployment.yaml
                        git add app-manifest/deployment.yaml
                        git commit -m "Update deployment image to version ${TAGVER}"
                        git push -f https://${GITHUB_TOKEN}@github.com/Duocpv0101/nginx-argocd.git HEAD:main
                    '''
                }
            }
        }
        stage('Remove Unused docker image') {
            steps{
                sh "docker rmi ${REGISTRY}:${NEWTAG}"
            }
        }
    }
}