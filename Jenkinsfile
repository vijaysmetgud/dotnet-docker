pipeline {
    agent any  // Run on any available agent
    
    environment {
        // Define environment variables if necessary
        DOCKER_IMAGE_NAME = 'my-dotdocker-app'
        DOCKER_TAG = 'latest'
        DOCKER_REGISTRY = 'vsmetgud/dotnet'  // Specify your Docker registry if needed
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout code from the repository
                    git branch: 'main', url: 'https://github.com/vijaysmetgud/dotnet-docker.git'
                }
            }
        }
        
        stage('Restore Dependencies') {
    steps {
        script {
            // Navigate to the folder containing the .csproj or .sln file
            dir('/var/lib/jenkins/workspace/dotdocker/samples/aspnetapp/') { // replace 'src' with the correct directory
                sh 'dotnet restore'
            }
        }
    }
}

        

        stage('Build') {
            steps {
                script {
                    // Build the .NET 8.0 application
                    sh 'dotnet build --configuration Release'
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    // Publish the application to a folder for Dockerization
                    sh 'dotnet publish --configuration Release --output ./publish'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image using Dockerfile
                    sh '''
                        docker build -t $DOCKER_IMAGE_NAME:$DOCKER_TAG .
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Run Docker container locally (for testing)
                    sh '''
                        docker run -d -p 8000:8000 --name $DOCKER_IMAGE_NAME $DOCKER_IMAGE_NAME:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main'  // Only push to Docker registry on the 'main' branch
            }
            steps {
                script {
                    // Push Docker image to registry (if configured)
                    if (DOCKER_REGISTRY != '') {
                        sh '''
                            docker tag $DOCKER_IMAGE_NAME:$DOCKER_TAG $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$DOCKER_TAG
                            docker push $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$DOCKER_TAG
                        '''
                    }
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                script {
                    // Remove the Docker container after running
                    sh 'docker rm -f $DOCKER_IMAGE_NAME'
                    // Optionally, remove the image after push (if you don't need it locally)
                    sh 'docker rmi $DOCKER_IMAGE_NAME:$DOCKER_TAG'
                }
            }
        }
    }

    post {
        always {
            // Clean up the Docker container if it exists
            sh 'docker ps -aq --filter name=$DOCKER_IMAGE_NAME | xargs -I {} docker rm -f {} || true'
        }

        success {
            echo "Build and Dockerize process completed successfully!"
        }

        failure {
            echo "Build or Dockerize project failed"
        }
    } 
}  
