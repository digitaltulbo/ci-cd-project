pipeline{
    agent any

    environment {
        dockerHubRegistry = 'digitaltulbo'
        dockerHubRegistryCredential = 'docker-hub'
        githubCredential = 'k8s_manifest_git'
    }

    stages {
        stage('check out application git branch'){
            steps {
                checkout scm
            }
            post {
                failure {
                    echo 'repository checkout failure'
                }
                success {
                    echo 'repository checkout success'
                }
            }
        }

        stage('docker image build'){
            steps{
                sh "docker build -t ${dockerHubRegistry}/frontend:${currentBuild.number} ./frontend"
                sh "docker build -t ${dockerHubRegistry}/frontend:latest ./frontend"

                sh "docker build -t ${dockerHubRegistry}/backend:${currentBuild.number} ./backend"
                sh "docker build -t ${dockerHubRegistry}/backend:latest ./backend"

                sh "docker build -t ${dockerHubRegistry}/mysql:${currentBuild.number} ./mysql"
                sh "docker build -t ${dockerHubRegistry}/mysql:latest ./mysql"
            }
            post {
                    failure {
                      echo 'Docker image build failure !'
                    }
                    success {
                      echo 'Docker image build success !'
                    }
            }
        }
        stage('Docker Image Push') {
            steps {
                withDockerRegistry([ credentialsId: dockerHubRegistryCredential, url: "" ]) {
                    sh "docker push ${dockerHubRegistry}/frontend:${currentBuild.number}"
                    sh "docker push ${dockerHubRegistry}/frontend:latest"

                    sh "docker push ${dockerHubRegistry}/backend:${currentBuild.number}"
                    sh "docker push ${dockerHubRegistry}/backend:latest"

                    sh "docker push ${dockerHubRegistry}/mysql:${currentBuild.number}"
                    sh "docker push ${dockerHubRegistry}/mysql:latest"

                    sleep 10 /* Wait uploading */
                }
            }
            post {
                    failure {
                      echo 'Docker Image Push failure !'
                      sh "docker rmi ${dockerHubRegistry}/frontend:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/frontend:latest"

                      sh "docker rmi ${dockerHubRegistry}/backend:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/backend:latest"

                      sh "docker rmi ${dockerHubRegistry}/mysql:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/mysql:latest"
                    }
                    success {
                      echo 'Docker image push success !'
                      sh "docker rmi ${dockerHubRegistry}/frontend:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/frontend:latest"

                      sh "docker rmi ${dockerHubRegistry}/backend:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/backend:latest"

                      sh "docker rmi ${dockerHubRegistry}/mysql:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/mysql:latest"
                    }
            }
        }
        stage('K8S Manifest Update') {
            steps {
                sh "ls"
                sh 'mkdir -p gitOpsRepo'
                dir("gitOpsRepo")
                {
                    git branch: "main",
                    credentialsId: githubCredential,
                    url: 'https://github.com/digitaltulbo/ci-cd-project.git'
                    sh "git config --global user.email 'djzepssa@gmail.com'" 
                    sh "git config --global user.name 'digitaltulbo'"

                    sh "sed -i 's/frontend:.*\$/frontend:${currentBuild.number}/g' react-frontend.yaml"
                    sh "sed -i 's/backend:.*\$/backend:${currentBuild.number}/g' nodejs-backend.yaml"
                    sh "sed -i 's/mysql:.*\$/mysql:${currentBuild.number}/g' mysql-deployment.yaml"

                    sh "git add ."
                    sh "git commit -m '[UPDATE] k8s ${currentBuild.number} image versioning'"
//                     sshagent(credentials: ['19bdc43b-f3be-4cb9-aa1d-9896f503e3e8']) {
//                         sh "git remote set-url origin git@github.com:digitaltulbo/ci-cd-project.git"
//                         sh "git push -u origin main"
//                     }
                    withCredentials([gitUsernamePassword(credentialsId: githubCredential,
                                     gitToolName: 'git-tool')]) {
                        sh "git remote set-url origin https://github.com/digitaltulbo/ci-cd-project.git"
                        sh "git push -u origin main"
                    }
                }
            }
            post {
                    failure {
                      echo 'K8S Manifest Update failure !'
                    }
                    success {
                      echo 'K8S Manifest Update success !'
                    }
            }
        }

    }
}
