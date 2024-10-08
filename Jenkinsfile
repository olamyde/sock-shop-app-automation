pipeline {
    agent any
    parameters {
        choice(
            name: 'CARTS_IMAGE', 
            choices: ['weaveworksdemos/carts', 'weaveworksdemos/carts-alternate'], 
            description: 'Select the image for the Carts service'
        )
        choice(
            name: 'MONGO_CARTS_IMAGE', 
            choices: ['mongo', 'mongo:4.0'], 
            description: 'Select the image for MongoDB (Carts)'
        )
        choice(
            name: 'CATALOGUE_IMAGE', 
            choices: ['weaveworksdemos/catalogue', 'weaveworksdemos/catalogue-alternate'], 
            description: 'Select the image for the Catalogue service'
        )
        choice(
            name: 'CATALOGUE_DB_IMAGE', 
            choices: ['weaveworksdemos/catalogue-db', 'weaveworksdemos/catalogue-db-alternate'], 
            description: 'Select the image for the Catalogue DB service'
        )
        choice(
            name: 'FRONT_END_IMAGE', 
            choices: ['weaveworksdemos/front-end', 'weaveworksdemos/front-end-alternate'], 
            description: 'Select the image for the Front-End service'
        )
        choice(
            name: 'ORDERS_IMAGE', 
            choices: ['weaveworksdemos/orders', 'weaveworksdemos/orders-alternate'], 
            description: 'Select the image for the Orders service'
        )
        choice(
            name: 'MONGO_ORDERS_IMAGE', 
            choices: ['mongo', 'mongo:4.0'], 
            description: 'Select the image for MongoDB (Orders)'
        )
        choice(
            name: 'PAYMENT_IMAGE', 
            choices: ['weaveworksdemos/payment', 'weaveworksdemos/payment-alternate'], 
            description: 'Select the image for the Payment service'
        )
        choice(
            name: 'QUEUE_MASTER_IMAGE', 
            choices: ['weaveworksdemos/queue-master', 'weaveworksdemos/queue-master-alternate'], 
            description: 'Select the image for the Queue Master service'
        )
        choice(
            name: 'RABBITMQ_IMAGE', 
            choices: ['rabbitmq', 'rabbitmq:3.6.8-management'], 
            description: 'Select the image for RabbitMQ'
        )
        choice(
            name: 'REDIS_IMAGE', 
            choices: ['redis', 'redis:alpine'], 
            description: 'Select the image for Redis'
        )
        choice(
            name: 'SHIPPING_IMAGE', 
            choices: ['weaveworksdemos/shipping', 'weaveworksdemos/shipping-alternate'], 
            description: 'Select the image for the Shipping service'
        )
        choice(
            name: 'USER_IMAGE', 
            choices: ['weaveworksdemos/user', 'weaveworksdemos/user-alternate'], 
            description: 'Select the image for the User service'
        )
        choice(
            name: 'USER_DB_IMAGE', 
            choices: ['weaveworksdemos/user-db', 'weaveworksdemos/user-db-alternate'], 
            description: 'Select the image for the User DB service'
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
                    // Collect selected images with tag applied
                    def images = [
                        'carts': "${params.CARTS_IMAGE}:${VERSION_TAG}",
                        'mongo-carts': params.MONGO_CARTS_IMAGE,
                        'catalogue': "${params.CATALOGUE_IMAGE}:${VERSION_TAG}",
                        'catalogue-db': "${params.CATALOGUE_DB_IMAGE}:${VERSION_TAG}",
                        'front-end': "${params.FRONT_END_IMAGE}:${VERSION_TAG}",
                        'orders': "${params.ORDERS_IMAGE}:${VERSION_TAG}",
                        'mongo-orders': params.MONGO_ORDERS_IMAGE,
                        'payment': "${params.PAYMENT_IMAGE}:${VERSION_TAG}",
                        'queue-master': "${params.QUEUE_MASTER_IMAGE}:${VERSION_TAG}",
                        'rabbitmq': params.RABBITMQ_IMAGE,
                        'redis': params.REDIS_IMAGE,
                        'shipping': "${params.SHIPPING_IMAGE}:${VERSION_TAG}",
                        'user': "${params.USER_IMAGE}:${VERSION_TAG}",
                        'user-db': "${params.USER_DB_IMAGE}:${VERSION_TAG}"
                    ]

                    // Update each Kubernetes manifest file with the corresponding image
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
