pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')      // Sonar token
        IMAGE = "devops"              // Docker Image Name
    }

    stages {

        /* ------------------------------
           GIT CHECKOUT
        --------------------------------*/
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Gittyuser4/DevSecOps-Connect-Four-Deployment.git'
            }
        }

        /* ------------------------------
           SONARQUBE SCAN
        --------------------------------*/
        stage('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {   
                        def scannerHome = tool 'sonar-scanner'

                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=connect-four \
                            -Dsonar.projectName=connect-four \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        /* ------------------------------
           QUALITY GATE
        --------------------------------*/
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        echo "Checking SonarQube Quality Gate..."
                        def qg = waitForQualityGate()
                        echo "Quality Gate Status: ${qg.status}"
                        if (qg.status != 'OK') {
                            error "Pipeline failed: Quality Gate status = ${qg.status}"
                        }
                    }
                }
            }
        }

        /* ------------------------------
           DOCKER BUILD
        --------------------------------*/
        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE}:v1.${BUILD_ID} .
                    docker tag ${IMAGE}:v1.${BUILD_ID} ${IMAGE}:latest
                '''
            }
        }

        /* ------------------------------
           DOCKER LOGIN
        --------------------------------*/
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker',
                                                  usernameVariable: 'USER',
                                                  passwordVariable: 'PASS')]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                }
            }
        }

        /* ------------------------------
           DOCKER PUSH
        --------------------------------*/
        stage('Docker Push') {
            steps {
                sh '''
                    docker push ${IMAGE}:v1.${BUILD_ID}
                    docker push ${IMAGE}:latest
                '''
            }
        }
    }
}
