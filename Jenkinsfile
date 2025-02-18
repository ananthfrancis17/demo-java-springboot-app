pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK11'
        jfrog 'jfrog-cli'
    }
    
    environment {
        SONAR_CREDENTIALS = credentials('sonar-token')
        ARTIFACTORY_CREDENTIALS = credentials('artifactory-credentials')
        DEPLOY_SERVER = '10.0.1.71' // Replace with your server IP
        DEPLOY_USER = 'ssm-user'     // Replace with your server username
        DEPLOY_PATH = '/home/ssm-user/application1' // Replace with your target directory
        SSH_CREDENTIALS = credentials('springboot-ssh') 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=hello-world \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_CREDENTIALS}
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Publish Artifacts'){
            steps {
                jf 'rt u target/*.jar springboot/'
            }
        }
        stage('Deploy to Server') {
            steps {
                script {
                    // Create target directory if it doesn't exist
                    sshagent(['springboot-ssh']) {
                        sh """
                            # Create directory if it doesn't exist
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} "mkdir -p ${DEPLOY_PATH}"
                            
                            # Copy the JAR file
                            scp -o StrictHostKeyChecking=no target/*.jar ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/
                            
                            # Optional: Restart service if needed
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} "nohup java -jar ${DEPLOY_PATH}/hello-world-1.0.0.jar > output.log 2>&1 &"
                        """
                    }
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