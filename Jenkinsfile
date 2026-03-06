pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = "http://localhost:8082/artifactory/eshop-generic-local"
        DEPLOY_SERVER = "192.168.56.110"
        DEPLOY_SHARE = "\\\\192.168.56.110\\c$\\temp"
        IIS_PATH = "C:\\inetpub\\Eshop"
    }

    stages {

        stage('Generate Timestamp') {
            steps {
                script {
                    env.BUILD_TIME = new Date().format("yyyyMMdd-HHmm")
                }
            }
        }

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

        stage('Remove Config Files') {
            steps {
                bat '''
                del publish\\appsettings*.json
                del publish\\web.config
                '''
            }
        }

        stage('Create Artifact') {
            steps {
                bat "powershell Compress-Archive -Path publish\\* -DestinationPath eshop-%BUILD_TIME%.zip"
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    bat """
                    curl -u %ART_USER%:%ART_PASS% ^
                    -T eshop-%BUILD_TIME%.zip ^
                    %ARTIFACTORY_URL%/eshop-%BUILD_TIME%.zip
                    """
                }
            }
        }

        stage('Download Artifact from Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    bat """
                    curl -u %ART_USER%:%ART_PASS% ^
                    -O %ARTIFACTORY_URL%/eshop-%BUILD_TIME%.zip
                    """
                }
            }
        }

        stage('Copy Artifact to Deployment Server') {
            steps {
                bat """
                copy eshop-%BUILD_TIME%.zip %DEPLOY_SHARE%
                """
            }
        }

        stage('Extract Artifact on Server') {
            steps {
                bat """
                powershell Invoke-Command -ComputerName %DEPLOY_SERVER% -ScriptBlock {
                    Expand-Archive -Path C:\\temp\\eshop-%BUILD_TIME%.zip -DestinationPath C:\\Inetpub\\Eshop -Force
                }
                """
            }
        }

        stage('Restart IIS') {
            steps {
                bat """
                powershell Invoke-Command -ComputerName %DEPLOY_SERVER% -ScriptBlock {
                    iisreset
                }
                """
            }
        }

    }
}
