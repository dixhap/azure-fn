pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS = credentials('AZURE_CREDENTIALS') // Jenkins credential with SPN info
        AZURE_FUNCTIONAPP_NAME = 'my-function-app'
        AZURE_RESOURCE_GROUP = 'my-resource-group'
        AZURE_REGION = 'westeurope'
        NODE_VERSION = '18'
        DB_NAME = 'myAppDatabase'
        ARTIFACT_NAME = 'functionapp.zip'
    }

    stages {
        stage('Install Node.js & Dependencies') {
            steps {
                // script {
                 //   sh "nvm install ${env.NODE_VERSION}"
                  //  sh "nvm use ${env.NODE_VERSION}"
               // }
                sh 'npm install'
            }
        }

        stage('Build (Optional)') {
            steps {
                sh 'npm run build || echo "No build step defined"'
            }
        }

        stage('Create Deployment Artifact') {
            steps {
                sh """
                    mkdir -p artifact
                    zip -r artifact/${env.ARTIFACT_NAME} * -x 'artifact/*' '.git/*' 'node_modules/*'
                """
                archiveArtifacts artifacts: "artifact/${env.ARTIFACT_NAME}", fingerprint: true
            }
        }

        stage('Azure Login') {
            steps {
                script {
                    sh """
                        az login --service-principal \\
                            -u ${env.AZURE_CREDENTIALS_USR} \\
                            -p ${env.AZURE_CREDENTIALS_PSW} \\
                            --tenant ${env.AZURE_CREDENTIALS_TEN}
                        az account set --subscription ${env.AZURE_CREDENTIALS_SUB}
                    """
                }
            }
        }

        stage('Update App Settings') {
            steps {
                sh """
                    az functionapp config appsettings set \\
                        --name ${env.AZURE_FUNCTIONAPP_NAME} \\
                        --resource-group ${env.AZURE_RESOURCE_GROUP} \\
                        --settings DB_NAME=${env.DB_NAME}
                """
            }
        }

        stage('Deploy ZIP Artifact to Azure Function App') {
            steps {
                sh """
                    az functionapp deployment source config-zip \\
                        --name ${env.AZURE_FUNCTIONAPP_NAME} \\
                        --resource-group ${env.AZURE_RESOURCE_GROUP} \\
                        --src artifact/${env.ARTIFACT_NAME}
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
        }
    }
}
