pipeline {
    agent any
    parameters {
        choice(
            name: 'SERVICE_IMAGE', 
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
            description: 'Select the base image for deployment across all services'
        )
        string(
            name: 'VERSION_TAG', 
            defaultValue: '1.0.0 or build-123abc', 
            description: 'Tag for the Docker image (leave blank to auto-generate using build number and commit hash)'
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
                    // Map each service to the selected image, adding the version tag
                    def images = [
                        'carts': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'mongo-carts': params.SERVICE_IMAGE,
                        'catalogue': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'catalogue-db': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'front-end': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'orders': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'mongo-orders': params.SERVICE_IMAGE,
                        'payment': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'queue-master': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'rabbitmq': params.SERVICE_IMAGE,
                        'redis': params.SERVICE_IMAGE,
                        'shipping': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'user': "${params.SERVICE_IMAGE}:${VERSION_TAG}",
                        'user-db': "${params.SERVICE_IMAGE}:${VERSION_TAG}"
                    ]

                    // Update each Kubernetes manifest file with the selected image and version tag
                    images.each { service, image ->
                        sh "sed -i 's|image: ${service}.*|image: ${image}|g' k8s/${service}-deployment.yaml"
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
