pipeline {

    agent none

    environment {
        ARTIFACTORY_URL = "http://192.168.56.1:8082/artifactory/eshop-generic-local"
    }

    stages {

        stage('Build Application') {

            agent { label 'windows-build-agent' }

            stages {

                stage('Checkout Code') {
                    steps {
                        checkout scm
                    }
                }

                stage('Restore Packages') {
                    steps {
                        bat 'dotnet restore src\\Web\\Web.csproj'
                    }
                }

                stage('Build Application') {
                    steps {
                        bat 'dotnet build src\\Web\\Web.csproj -c Release --no-restore'
                    }
                }

                stage('Publish Application') {
                    steps {
                        bat 'dotnet publish src\\Web\\Web.csproj -c Release -o publish --no-build'
                    }
                }

                stage('Prepare Files') {
                    steps {
                        bat '''
                        powershell Remove-Item publish\\web.config -ErrorAction SilentlyContinue
                        powershell Remove-Item publish\\appsettings.json -ErrorAction SilentlyContinue
                        '''
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

                stage('Deploy') {

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
