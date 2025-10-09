pipeline {
    agent any

    environment {
        ACR_NAME = 'perseverance01'
        RESOURCE_GROUP = 'dev-env'
        AKS_CLUSTER = 'dev-cluster'
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/rohanjob/Dataabase.git'
            }
        }

        stage('Login to Azure') {
            steps {
                sh '''
                az login --service-principal \
                    -u $AZURE_CLIENT_ID \
                    -p $AZURE_CLIENT_SECRET \
                    --tenant $AZURE_TENANT_ID
                az account set --subscription $(az account show --query id -o tsv)
                '''
            }
        }

        stage('Login to ACR') {
            steps {
                sh '''
                az acr login --name $ACR_NAME
                '''
            }
        }

        stage('Build & Push Images') {
            steps {
                sh '''
                docker build -t $ACR_NAME.azurecr.io/backend:$BUILD_TAG ./backend
                docker push $ACR_NAME.azurecr.io/backend:$BUILD_TAG

                docker build -t $ACR_NAME.azurecr.io/frontend:$BUILD_TAG ./frontend
                docker push $ACR_NAME.azurecr.io/frontend:$BUILD_TAG
                '''
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh '''
                az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER --overwrite-existing

                kubectl set image deployment/backend backend=$ACR_NAME.azurecr.io/backend:$BUILD_TAG -n three-tier --record || true
                kubectl set image deployment/frontend frontend=$ACR_NAME.azurecr.io/frontend:$BUILD_TAG -n three-tier --record || true

                kubectl rollout status deployment/backend -n three-tier
                kubectl rollout status deployment/frontend -n three-tier
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Successfully deployed build number ${BUILD_TAG}!"
        }
        failure {
            echo "❌ Deployment failed. Check logs for details."
        }
    }
}
