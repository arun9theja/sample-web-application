def getDockerTag(){
    def tag = sh script: 'git rev-parse --short HEAD', returnStdout: true
    return tag
    }

pipeline{
    agent any
    environment{
        Docker_tag = getDockerTag()
        docker_pws = credentials('docker-hub-password')
    }
    stages{
        stage("Sonar scan"){
            agent {
                docker {
                    image 'maven'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps{
                script{
                   sh "echo executing sonar scan"
                   // sh "sleep 90"
                }
            }
        }

        stage('build the application'){
            agent {
                docker {
                    image 'maven'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps{
                script{
                    sh "mvn clean install"
                }
            }
        }

        stage('docker build'){
            steps{
                script{
                    sh """
                    docker build . -t deekshithsn/java-app-hbc:$Docker_tag
                    """
                }
            }
        }

        
        stage('docker login & push'){
            steps{
                script{
                    sh """
                       docker login -u deekshithsn -p $docker_pws 
                       docker push deekshithsn/java-app-hbc:$Docker_tag
                    """
                    addBadge(icon: 'save.gif', text: 'docker repo', link: 'https://hub.docker.com/repository/docker/deekshithsn/java-app-hbc')
                    currentBuild.description = "deekshithsn/java-app-hbc:$Docker_tag"
                }
            }
        }

        stage('deploy the application'){
            steps{
                script{
                    configFileProvider([configFile(fileId: 'kube-dev-config', variable: 'KUBECONFIG')]) {
                        sh '''
                            kubectl get po
                            final_tag=$(echo $Docker_tag | tr -d ' ')
                            sed -i "s|TAG|$final_tag|" deployment.yaml
                            cat deployment.yaml
                        '''
                    }
                }
            }
        }

    }

    post {
        always{
            cleanWs()
        }
    }
}
