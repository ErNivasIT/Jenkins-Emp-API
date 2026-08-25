pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        
        stage('Restore & Build') {
            steps {
                sh 'dotnet restore'
                sh 'dotnet build --configuration Release --no-restore'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                // Builds a docker image tagged as bookstore-api:latest
                sh 'docker build -t bookstore-api:latest -f BookStore.Api/Dockerfile .'
            }
        }

        stage('Deploy / Run Container') {
            steps {
                // Stop and remove any old container running on port 8083 to avoid port conflict
                sh 'docker stop bookstore-api-container || true'
                sh 'docker rm bookstore-api-container || true'
                
                // Run the new container mapping port 8083 on your host to port 8080 inside the container
                sh 'docker run -d -p 8083:8080 --name bookstore-api-container bookstore-api:latest'
            }
        }
    }
}