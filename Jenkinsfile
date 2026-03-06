pipeline {

    agent { label 'windows-build-agent' }

    environment {

        ARTIFACTORY_URL = "http://192.168.56.1:8082/artifactory/eshop-generic-local"
        DEPLOY_SERVER = "192.168.56.113"
        IIS_PATH = "C:\\inetpub\\eshop"
        TEMP_PATH = "C:\\temp"

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
                    curl -u %USER%:%PASS% -T %ZIP_NAME% %ARTIFACTORY_URL%/%ZIP_NAME%
                    '''

                }

            }

        }

    }

}
