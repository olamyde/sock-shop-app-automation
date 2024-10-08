pipeline {
    agent any
    parameters {
        choice(
            name: 'IMAGE_NAME', 
            choices: [
                'weaveworksdemos/carts',
                'mongo',
                'weaveworksdemos/catalogue',
                'weaveworksdemos/catalogue-db',
                'weaveworksdemos/front-end',
                'weaveworksdemos/orders',
                'weaveworksdemos/payment',
                'weaveworksdemos/queue-master',
                'rabbitmq',
                'redis',
                'weaveworksdemos/shipping',
                'weaveworksdemos/user',
                'weaveworksdemos/user-db'
            ], 
            description: 'Select the base image for the Docker image (specific to each service)'
        )
        string(
            name: 'VERSION_TAG', 
            defaultValue: '1.0.0 or build-123abc', // Sample format for guidance
            description: 'Tag for the Docker image (e.g., 1.0.0, build-123abc; leave blank to auto-generate using build number and commit hash)'
        )
    }
    environment {
        KUBECONFIG = credentials('kubeconfig') // Replace with your Jenkins credential ID for Kubeconfig
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/olamyde/sock-shop-app-automation.git' // Replace with your repo URL
            }
        }

        stage('Set Version Tag') {
            steps {
                script {
                    // If VERSION_TAG parameter is not set, generate one using the build number and commit hash
                    if (!params.VERSION_TAG) {
                        def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                        env.VERSION_TAG = "${BUILD_NUMBER}-${commitHash}"
                    } else {
                        env.VERSION_TAG = params.VERSION_TAG
                    }
                    echo "Using version tag: ${VERSION_TAG}"
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Define base images for each service
                    def defaultImages = [
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

                    // Use the IMAGE_NAME parameter for selected image or default images for each service
                    def images = defaultImages.collectEntries { service, baseImage ->
                        [(service): params.IMAGE_NAME ? "${params.IMAGE_NAME}" : baseImage]
                    }

                    // Iterate through the images map and apply the VERSION_TAG
                    images.each { service, baseImage ->
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
