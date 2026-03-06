pipeline {
    agent any

    environment {
        SERVER_IP = "192.168.56.110"
        DEPLOY_PATH = "C:\\inetpub\\eshop"
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
                    http://localhost:8082/artifactory/eshop-generic-local/eshop-%BUILD_TIME%.zip
                    '''
                }
            }
        }

        stage('deploy to windows server') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'server-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    powershell """
                    \$secpass = ConvertTo-SecureString '${PASS}' -AsPlainText -Force
                    \$cred = New-Object System.Management.Automation.PSCredential('${USER}', \$secpass)

                    Invoke-Command -ComputerName ${SERVER_IP} -Credential \$cred -ScriptBlock {

                        if(!(Test-Path 'C:\\temp')){
                            New-Item -ItemType Directory -Path 'C:\\temp'
                        }

                        curl -o C:\\temp\\eshop.zip http://192.168.56.1:8082/artifactory/eshop-generic-local/eshop-${BUILD_TIME}.zip

                        Import-Module WebAdministration
                        Stop-WebAppPool -Name "eshop"

                        Expand-Archive -Path C:\\temp\\eshop.zip -DestinationPath ${DEPLOY_PATH} -Force

                        Start-WebAppPool -Name "eshop"
                    }
                    """
                }
            }
        }

    }
}
