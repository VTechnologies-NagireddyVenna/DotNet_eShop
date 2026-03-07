pipeline {

    agent none

    environment {
        ARTIFACTORY_URL = "http://192.168.56.1:8082/artifactory/eshop-generic-local"
    }

    stages {

        stage('Build & Package') {
            agent { label 'windows-build-agent' }

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
                        bat 'dotnet publish src\\Web\\Web.csproj -c Release -o publish'
                    }
                }

                stage('Create ZIP') {
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

                stage('Upload Artifact to JFrog') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {

                            bat '''
                            curl.exe -u %USER%:%PASS% -T %ZIP_NAME% %ARTIFACTORY_URL%/%ZIP_NAME%
                            '''

                        }
                    }
                }

            }
        }

        stage('Deploy to IIS Server') {
            agent { label 'deploy-agent' }

            stages {

                stage('Download Artifact') {
                    steps {

                        withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {

                            bat '''
                            curl.exe -L -u %USER%:%PASS% -O %ARTIFACTORY_URL%/%ZIP_NAME%
                            '''

                        }

                    }
                }

                stage('Deploy Application') {
                    steps {

                        bat '''
                        powershell -Command "if(Test-Path 'C:\\inetpub\\eshop'){Remove-Item 'C:\\inetpub\\eshop\\*' -Recurse -Force}"
                        '''

                        bat '''
                        powershell Expand-Archive %ZIP_NAME% C:\\inetpub\\eshop -Force
                        '''

                    }
                }

                stage('Restart IIS AppPool') {
                    steps {

                        bat '''
                        powershell Import-Module WebAdministration
                        '''

                        bat '''
                        powershell Restart-WebAppPool -Name eshop
                        '''

                    }
                }

            }
        }

    }

}
