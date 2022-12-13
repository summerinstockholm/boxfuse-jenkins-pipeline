pipeline {

    agent any

    stages {
        stage('Fetch and maven build') {
            agent {
                docker {
                    image "84.201.177.137:8123/docker-jenkins-mvn-agent:${params.builderVersion}"
                    args '-v /var/run/docker.sock:/var/run/docker.sock  -u root:root'
                    registryUrl 'http://84.201.177.137:8123/'
                    registryCredentialsId 'nexus-cred'
                    reuseNode true
                }
            }
            steps {
                git 'https://github.com/summerinstockholm/boxfuse-sample-cicd'
                sh "mvn clean package"
            }
        }
        stage('Get docker socket group') {
            steps {
                script {
                    DOCKER_GROUP = sh(returnStdout: true, script: 'stat -c %g /var/run/docker.sock').trim()
                }
            }
        }
        stage('Docker build, push and local remove') {
            agent {
                docker {
                    image "84.201.177.137:8123/docker-jenkins-mvn-agent:${params.builderVersion}"
                    registryUrl 'http://84.201.177.137:8123/'
                    registryCredentialsId 'nexus-cred'
                    args "-v /var/run/docker.sock:/var/run/docker.sock --group-add ${DOCKER_GROUP}"
                    reuseNode true
                }
            }
            steps {
                sh "echo FROM 84.201.177.137:8123/app-template:${params.templateAppVersion} > Dockerfile"
                sh "echo RUN rm -rf /opt/tomcat/webapps/ROOT >> Dockerfile"
                sh "echo COPY ./target/*.war /opt/tomcat/webapps/ROOT.war >> Dockerfile"
                sh "docker build -t 84.201.177.137:8123/boxfuse-app:${params.appVersion} ."
                sh "docker push 84.201.177.137:8123/boxfuse-app:${params.appVersion}"
                sh "docker rmi 84.201.177.137:8123/boxfuse-app:${params.appVersion}"
            }
        }
        stage('Deploy at yc-deploy-host') {
            steps {
                sshagent(['cbd47a70-263b-48e0-bcfc-b9134f91704b']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no -l root 84.201.139.208 'sh -s << "ENDSH"
                    for ID in \$(docker ps -q); do docker stop \$ID; done
                    for ID in \$(docker ps -a -q); do docker rm \$ID; done
                    for ID in \$(docker images -q); do docker rmi \$ID; done
                    docker pull 84.201.177.137:8123/boxfuse-app:${params.appVersion}
                    docker run -p 80:8080 -d --name boxfuse-app 84.201.177.137:8123/boxfuse-app:${params.appVersion}
ENDSH\'
                    """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}