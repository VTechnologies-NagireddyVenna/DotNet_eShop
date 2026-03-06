pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = "http://localhost:8082/artifactory/eshop-generic-local"
    }

    stages {

        stage('Restore') {
            steps {
                bat 'dotnet restore eShopOnWeb.sln'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build eShopOnWeb.sln --configuration Release'
            }
        }

        stage('Publish') {
            steps {
                bat 'dotnet publish src/Web/Web.csproj -c Release -o publish'
            }
        }

        stage('Create Artifact') {
            steps {
                bat 'powershell Compress-Archive -Path publish\\* -DestinationPath eshop-%BUILD_NUMBER%.zip'
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    bat """
                    curl -u %ART_USER%:%ART_PASS% ^
                    -T eshop-%BUILD_NUMBER%.zip ^
                    %ARTIFACTORY_URL%/eshop-%BUILD_NUMBER%.zip
                    """
                }
            }
        }

    }
}
