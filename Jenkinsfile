pipeline {
    agent any

    environment {
        PROJECT_ID = 'intense-acrobat-453805-m5'           // ✅ CHANGE: your actual GCP project ID
        IMAGE_NAME = 'my-node-app'                          // ✅ CHANGE: your app name/image
        GCR_IMAGE = "gcr.io/${PROJECT_ID}/${IMAGE_NAME}"
        CREDENTIALS_ID = 'gcp-credentials'                  // ✅ CHANGE: Jenkins secret file ID (see below)
        CLUSTER_NAME = 'demo-cluster'                       // ✅ CHANGE: your GKE cluster name
        CLUSTER_ZONE = 'asia-south1-c'                      // ✅ CHANGE: zone where your GKE is deployed
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo '🔄 Cloning GitHub repo...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image: ${GCR_IMAGE}"
                sh "docker build -t ${GCR_IMAGE} ."
            }
        }

        stage('Authenticate with GCP and Push Image') {
            steps {
                echo '🔐 Authenticating with GCP...'
                withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh 'gcloud auth configure-docker --quiet'
                    echo '🚀 Pushing image to Google Container Registry...'
                    sh "docker push ${GCR_IMAGE}"
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                echo "📦 Deploying to GKE cluster: ${CLUSTER_NAME}"
                sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${CLUSTER_ZONE} --project ${PROJECT_ID}"
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline completed successfully.'
        }
        failure {
            echo '❌ CI/CD pipeline failed.'
        }
    }
}

