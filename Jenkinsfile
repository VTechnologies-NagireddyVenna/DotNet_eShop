pipeline {
    agent any
    stages {
        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build --configuration Release'
            }
        }

        stage('Publish') {
            steps {
                bat 'dotnet publish src/Web/Web.csproj -c Release -o publish'
            }
        }

        stage('Create Artifact') {
            steps {
                bat 'powershell Compress-Archive -Path publish\\* -DestinationPath eshop.zip'
            }
        }
    }
}
