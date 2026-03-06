pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = "http://localhost:8082/artifactory/eshop-generic-local"
        DEPLOY_SERVER = "192.168.56.110"
        APP_POOL = "eshop"
    }

    stages {

        stage('generate timestamp') {
            steps {
                script {
                    env.BUILD_TIME = new Date().format("yyyyMMdd-HHmm")
                }
            }
        }

        stage('restore') {
            steps {
                bat 'dotnet restore eShopOnWeb.sln'
            }
        }

        stage('build') {
            steps {
                bat 'dotnet build eShopOnWeb.sln --configuration Release'
            }
        }

        stage('publish') {
            steps {
                bat 'dotnet publish src/Web/Web.csproj -c Release -o publish'
            }
        }

        stage('remove config files') {
            steps {
                bat '''
                del publish\\appsettings*.json
                del publish\\web.config
                '''
            }
        }

        stage('create artifact') {
            steps {
                bat 'powershell Compress-Archive -Path publish\\* -DestinationPath eshop-%BUILD_TIME%.zip'
            }
        }

        stage('upload artifact to artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat '''
                    curl -u %USER%:%PASS% ^
                    -T eshop-%BUILD_TIME%.zip ^
                    %ARTIFACTORY_URL%/eshop-%BUILD_TIME%.zip
                    '''
                }
            }
        }

        stage('download artifact from artifactory to server') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat '''
                    powershell Invoke-Command -ComputerName 192.168.56.110 -ScriptBlock {
                        curl -u %USER%:%PASS% -o C:\\temp\\eshop.zip http://192.168.56.1:8082/artifactory/eshop-generic-local/eshop-%BUILD_TIME%.zip
                    }
                    '''
                }
            }
        }

        stage('stop iis apppool') {
            steps {
                bat '''
                powershell Invoke-Command -ComputerName 192.168.56.110 -ScriptBlock {
                    Stop-WebAppPool -Name "eshop"
                }
                '''
            }
        }

        stage('extract artifact') {
            steps {
                bat '''
                powershell Invoke-Command -ComputerName 192.168.56.110 -ScriptBlock {
                    Expand-Archive -Path C:\\temp\\eshop.zip -DestinationPath C:\\inetpub\\eshop -Force
                }
                '''
            }
        }

        stage('start iis apppool') {
            steps {
                bat '''
                powershell Invoke-Command -ComputerName 192.168.56.110 -ScriptBlock {
                    Start-WebAppPool -Name "eshop"
                }
                '''
            }
        }

    }
}
