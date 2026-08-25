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
        
        stage('Run Tests & Coverage') {
            steps {
                sh '''
                    dotnet test BookStore.Tests/BookStore.Tests.csproj \
                        --configuration Release \
                        --no-build \
                        --collect:"XPlat Code Coverage" || true
                '''
                
                sh '''
                    dotnet reportgenerator \
                        -reports:"BookStore.Tests/TestResults/**/coverage.cobertura.xml" \
                        -targetdir:"coveragereport" \
                        -reporttypes:Html
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t bookstore-api:latest .'
            }
        }

        stage('Deploy / Run Container') {
            steps {
                sh 'docker stop bookstore-api-container || true'
                sh 'docker rm bookstore-api-container || true'
                sh 'docker run -d -p 8083:8080 --name bookstore-api-container bookstore-api:latest'
            }
        }
    }
}