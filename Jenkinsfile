pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '104360117424.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPOSITORY = 'project4-app'
        ECR_URI = '104360117424.dkr.ecr.ap-south-1.amazonaws.com/project4-app'

        GITHUB_REPO = 'https://github.com/shaunjohn-04/project4-devsecops.git'

        SONAR_HOST_URL = 'http://172.31.9.81:9000'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm

                echo 'Source code downloaded from GitHub'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'sonarqube-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    sh '''
                        docker run --rm \
                          --network host \
                          -e SONAR_HOST_URL="$SONAR_HOST_URL" \
                          -e SONAR_TOKEN="$SONAR_TOKEN" \
                          -v "$WORKSPACE/project4:/usr/src" \
                          sonarsource/sonar-scanner-cli:latest \
                          -Dsonar.projectKey=project4 \
                          -Dsonar.projectName="Project 4 DevSecOps" \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions="**/migrations/**,**/reports/**,**/__pycache__/**,**/templates/studentproedit.html"
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    env.IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                }

                sh '''
                    docker build \
                      -t $ECR_URI:$IMAGE_TAG \
                      -t $ECR_URI:latest \
                      project4/
                '''

                echo 'Docker image built successfully'
            }
        }

        stage('ECR Login and Push') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'AWS_ACCESS_KEY_ID',
                        variable: 'AWS_ACCESS_KEY_ID'
                    ),
                    string(
                        credentialsId: 'AWS_SECRET_ACCESS_KEY',
                        variable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        export AWS_DEFAULT_REGION=$AWS_REGION

                        aws sts get-caller-identity

                        aws ecr get-login-password \
                          --region $AWS_REGION | \
                          docker login \
                          --username AWS \
                          --password-stdin $ECR_REGISTRY

                        docker push $ECR_URI:$IMAGE_TAG
                        docker push $ECR_URI:latest
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'github-token',
                        variable: 'GITHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        sed -i \
                          "s|image: .*project4-app:.*|image: $ECR_URI:$IMAGE_TAG|" \
                          project4/k8s/app.yaml

                        echo "Updated Kubernetes image:"
                        grep "image:" project4/k8s/app.yaml

                        git config user.name "Jenkins"
                        git config user.email "jenkins@localhost"

                        git add project4/k8s/app.yaml

                        git commit -m "Deploy Project 4 image $IMAGE_TAG"

                        AUTH=$(printf 'x-access-token:%s' "$GITHUB_TOKEN" | base64 -w0)

                        git -c http.extraheader="AUTHORIZATION: basic $AUTH" \
                          push origin HEAD:main
                    '''
                }
            }
        }

    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo 'Argo CD will deploy the updated image to EKS.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }

        always {
            echo 'Project 4 pipeline execution finished.'
        }
    }
}
