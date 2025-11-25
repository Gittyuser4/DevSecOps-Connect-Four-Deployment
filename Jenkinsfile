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


        stage('Trivy Image Scan') {
            steps {
                sh '''
                    echo "Running Trivy scan on Docker image: ${IMAGE}:v1.${BUILD_ID}"
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image \
                        --exit-code 0 \
                        --severity HIGH,CRITICAL \
                        --ignore-unfixed ${IMAGE}:v1.${BUILD_ID}
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
