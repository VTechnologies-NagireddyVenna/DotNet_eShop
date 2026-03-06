pipeline {
    agent any

    environment {
        SERVER_IP = "192.168.56.110"
        ARTIFACTORY_URL = "http://localhost:8082/artifactory/eshop-generic-local"
        DEPLOY_PATH = "C:\\inetpub\\eshop"
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

        stage('Upload Artifact to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat """
                    curl -u %USER%:%PASS% ^
                    -T eshop-%BUILD_TIME%.zip ^
                    ${ARTIFACTORY_URL}/eshop-%BUILD_TIME%.zip
                    """
                }
            }
        }

        stage('Deploy to Windows Server') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'server-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {

                    powershell '''
                    $username = $env:USER
                    $password = $env:PASS | ConvertTo-SecureString -AsPlainText -Force
                    $cred = New-Object System.Management.Automation.PSCredential($username,$password)

                    Invoke-Command -ComputerName 192.168.56.110 -Credential $cred -Authentication Basic -ScriptBlock {

                        $artifactUrl = "http://192.168.56.1:8082/artifactory/eshop-generic-local/eshop.zip"
                        $tempPath = "C:\\temp\\eshop.zip"
                        $deployPath = "C:\\inetpub\\eshop"

                        if(!(Test-Path "C:\\temp")){
                            New-Item -ItemType Directory -Path "C:\\temp"
                        }

                        curl -o $tempPath $artifactUrl

                        Import-Module WebAdministration

                        if(Test-Path "IIS:\\AppPools\\eshop"){
                            Stop-WebAppPool -Name "eshop"
                        }

                        if(!(Test-Path $deployPath)){
                            New-Item -ItemType Directory -Path $deployPath
                        }

                        Expand-Archive -Path $tempPath -DestinationPath $deployPath -Force

                        Start-WebAppPool -Name "eshop"
                    }
                    '''
                }
            }
        }

    }

}
