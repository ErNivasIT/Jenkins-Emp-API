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
                // Run tests, collect code coverage in Cobertura format, and output results to a 'coverage' folder
                sh '''
                    dotnet test BookStore.Tests/BookStore.Tests.csproj \
                        --configuration Release \
                        --no-build \
                        --collect:"XPlat Code Coverage" \
                        --settings BookStore.Tests/coverlet.runsettings || true
                '''
                
                // Convert the XML coverage file into a professional HTML report using ReportGenerator
                // (Using the full path to ensure Jenkins finds the global tool)
                sh '''
                    /root/.dotnet/tools/reportgenerator \
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
    
    post {
        success {
            // Publish the HTML code coverage report to the Jenkins dashboard
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'coveragereport',
                reportFiles: 'index.html',
                reportName: 'Code Coverage Report',
                evalAllFiles: false
            ])
        }
    }
}