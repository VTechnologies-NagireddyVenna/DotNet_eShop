pipeline {

    agent { label 'windows-build-agent' }

    environment {

        ARTIFACTORY_URL = "http://192.168.56.1:8082/artifactory/eshop-generic-local"
        DEPLOY_SERVER = "192.168.56.113"
        IIS_PATH = "C:\\inetpub\\eshop"
        TEMP_PATH = "C:\\temp"

    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/VTechnologies-NagireddyVenna/DotNet_eShop.git'
            }
        }

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
                    curl -u %USER%:%PASS% -T %ZIP_NAME% %ARTIFACTORY_URL%/%ZIP_NAME%
                    '''

                }

            }

        }

        stage('Download Artifact on Deployment Server') {

            steps {

                withCredentials([usernamePassword(credentialsId: 'deploy-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {

                    powershell """

                    \$sec = ConvertTo-SecureString '$PASS' -AsPlainText -Force
                    \$cred = New-Object System.Management.Automation.PSCredential('$USER', \$sec)

                    Invoke-Command -ComputerName $env.DEPLOY_SERVER -Credential \$cred -ScriptBlock {

                        if(!(Test-Path '$env:TEMP_PATH')){
                            New-Item -ItemType Directory -Path '$env:TEMP_PATH'
                        }

                        \$url = "$env:ARTIFACTORY_URL/$env:ZIP_NAME"
                        \$output = "$env:TEMP_PATH\\$env:ZIP_NAME"

                        Invoke-WebRequest -Uri \$url -OutFile \$output

                    }

                    """

                }

            }

        }

        stage('Stop IIS App Pool') {

            steps {

                powershell """

                Invoke-Command -ComputerName $env.DEPLOY_SERVER -ScriptBlock {

                    Import-Module WebAdministration
                    Stop-WebAppPool -Name "DefaultAppPool"

                }

                """

            }

        }

        stage('Extract Artifact') {

            steps {

                powershell """

                Invoke-Command -ComputerName $env.DEPLOY_SERVER -ScriptBlock {

                    \$zip = "$env:TEMP_PATH\\$env:ZIP_NAME"
                    \$dest = "$env:IIS_PATH"

                    if(Test-Path \$dest){
                        Remove-Item \$dest\\* -Recurse -Force
                    }

                    Expand-Archive -Path \$zip -DestinationPath \$dest -Force

                }

                """

            }

        }

        stage('Start IIS App Pool') {

            steps {

                powershell """

                Invoke-Command -ComputerName $env.DEPLOY_SERVER -ScriptBlock {

                    Import-Module WebAdministration
                    Start-WebAppPool -Name "DefaultAppPool"

                }

                """

            }

        }

    }

}
