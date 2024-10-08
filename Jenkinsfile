pipeline {
    agent any
    environment {
        KUBECONFIG = credentials('kubeconfig') // Replace with your Jenkins credential ID for Kubeconfig
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Tankouatch/your-repo' // Replace with your repo URL
            }
        }

        stage('Set Version Tag') {
            steps {
                script {
                    // Generate a unique version tag using the build number and commit hash
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.VERSION_TAG = "${BUILD_NUMBER}-${commitHash}"
                    echo "Using version tag: ${VERSION_TAG}"
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Define base images for each service
                    def baseImages = [
                        'carts': 'weaveworksdemos/carts',
                        'mongo-carts': 'mongo',
                        'catalogue': 'weaveworksdemos/catalogue',
                        'catalogue-db': 'weaveworksdemos/catalogue-db',
                        'front-end': 'weaveworksdemos/front-end',
                        'orders': 'weaveworksdemos/orders',
                        'mongo-orders': 'mongo',
                        'payment': 'weaveworksdemos/payment',
                        'queue-master': 'weaveworksdemos/queue-master',
                        'rabbitmq': 'rabbitmq',
                        'redis': 'redis',
                        'shipping': 'weaveworksdemos/shipping',
                        'user': 'weaveworksdemos/user',
                        'user-db': 'weaveworksdemos/user-db'
                    ]

                    // Iterate through the base images map and apply the VERSION_TAG
                    baseImages.each { service, baseImage ->
                        def fullImage = baseImage.contains('mongo') || baseImage == 'rabbitmq' || baseImage == 'redis' ? baseImage : "${baseImage}:${VERSION_TAG}"
                        sh "sed -i 's|image: ${baseImage}.*|image: ${fullImage}|g' k8s/${service}-deployment.yaml"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(credentialsId: 'kubeconfig') {
                        // Apply all Kubernetes manifest files in the k8s directory
                        sh "kubectl apply -f k8s/"
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
