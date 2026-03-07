pipeline {

    agent none

    environment {
        ARTIFACTORY_URL = "http://192.168.56.1:8082/artifactory/eshop-generic-local"
    }

    stages {

        stage('Build') {

            agent { label 'windows-build-agent' }

            stages {

                stage('Checkout') {
                    steps {
                        checkout scm
                    }
                }

                stage('Restore') {
                    steps {
                        bat 'dotnet restore src\\Web\\Web.csproj'
                    }
                }

                stage('Build') {
                    steps {
                        bat 'dotnet build src\\Web\\Web.csproj --configuration Release --no-restore'
                    }
                }

                stage('Publish') {
                    steps {
                        bat 'dotnet publish src\\Web\\Web.csproj -c Release -o publish --no-build'
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

                stage('Upload Artifact') {

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

        stage('Deploy') {

            agent { label 'deploy-agent' }

            options { skipDefaultCheckout() }

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

                stage('Deploy to IIS') {

                    steps {

                        bat '''
                        powershell Stop-WebAppPool -Name eshop
                        '''

                        bat '''
                        powershell Remove-Item C:\\inetpub\\eshop\\* -Recurse -Force
                        '''

                        bat '''
                        powershell Expand-Archive %ZIP_NAME% C:\\inetpub\\eshop -Force
                        '''

                        bat '''
                        powershell Start-WebAppPool -Name eshop
                        '''

                    }

                }

            }

        }

    }

}
