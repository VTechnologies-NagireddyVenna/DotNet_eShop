pipeline {
    agent { label 'windows-build-agent' }

    environment {
        ARTIFACTORY_URL = "http://192.168.56.1:8082/artifactory/eshop-generic-local"
        DEPLOY_SERVER = "192.168.56.113"
        IIS_PATH = "C:\\inetpub\\eshop"
        APPPOOL = "eshop"
    }

    stages {

        stage('Restore Packages') {
            steps {
                bat 'dotnet restore eShopOnWeb.sln'
            }
        }

        stage('Build Application') {
            steps {
                bat 'dotnet build eShopOnWeb.sln --configuration Release'
            }
        }

        stage('Publish Application') {
            steps {
                bat 'dotnet publish src\\Web\\Web.csproj -c Release -o publish'
            }
        }

        stage('Create ZIP Artifact') {
            steps {
                script {
                    def timestamp = new Date().format("yyyyMMdd-HHmm")
                    env.ZIP_NAME = "eshop-${timestamp}.zip"
                }

                bat '''
                powershell Compress-Archive -Path publish\\* -DestinationPath %ZIP_NAME%
                '''
            }
        }

        stage('Upload Artifact to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat '''
                    curl.exe -u %USER%:%PASS% -T %ZIP_NAME% %ARTIFACTORY_URL%/%ZIP_NAME%
                    '''
                }
            }
        }

        stage('Download Artifact from Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat '''
                    curl.exe -L -u %USER%:%PASS% -O %ARTIFACTORY_URL%/%ZIP_NAME%
                    '''
                }
            }
        }

        stage('Copy Artifact to Deployment Server') {
            steps {
                bat '''
                copy %ZIP_NAME% \\\\%DEPLOY_SERVER%\\c$\\temp\\%ZIP_NAME%
                '''
            }
        }

        stage('Deploy to IIS Server') {
            steps {
                bat '''
                powershell -Command "Invoke-Command -ComputerName %DEPLOY_SERVER% -ScriptBlock {
                    param($zip)

                    $deployPath='C:\\inetpub\\eshop'
                    $zipPath='C:\\temp\\' + $zip

                    if(Test-Path $deployPath){
                        Remove-Item $deployPath\\* -Recurse -Force
                    }

                    Expand-Archive $zipPath -DestinationPath $deployPath -Force
                } -ArgumentList '%ZIP_NAME%'"
                '''
            }
        }

        stage('Restart IIS AppPool') {
            steps {
                bat '''
                powershell -Command "Invoke-Command -ComputerName %DEPLOY_SERVER% -ScriptBlock {
                    Import-Module WebAdministration
                    Restart-WebAppPool -Name 'eshop'
                }"
                '''
            }
        }

    }
}
