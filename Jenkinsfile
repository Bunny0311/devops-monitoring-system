pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        APP_NAME = 'myapp'
        NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code is available in the Jenkins workspace.'
                sh 'git --version'
            }
        }

        stage('Validate Kubernetes Manifests') {
            steps {
                sh '''
                    set -eu
                    test -f app/deployment.yaml
                    test -f app/service.yaml
                    test -f alerts/app-alerts.yaml
                    echo "All required Kubernetes manifests exist."
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    set -eu

                    kubectl apply -f app/deployment.yaml
                    kubectl apply -f app/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu

                    kubectl rollout status deployment/$APP_NAME \
                        -n $NAMESPACE --timeout=180s

                    kubectl get deployment $APP_NAME -n $NAMESPACE
                    kubectl get service myapp-service -n $NAMESPACE

                    AVAILABLE=$(kubectl get deployment $APP_NAME \
                        -n $NAMESPACE \
                        -o jsonpath='{.status.availableReplicas}')

                    if [ "$AVAILABLE" != "2" ]; then
                        echo "Expected 2 available replicas; found ${AVAILABLE:-0}"
                        exit 1
                    fi

                    echo "Deployment verification successful."
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
