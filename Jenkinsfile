pipeline {
    agent any

    environment {
        // Define your environment variables here
        DOCKER_BFLASK_IMAGE = "sayeda77/mypython:${BUILD_NUMBER}"
        ROLLBACK_IMAGE = "sayeda77/my-flask:previous"
        DOCKER_REGISTRY_CREDS = 'sayeda77/Dockerhub'
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t my-flask .'
                sh 'docker tag my-flask $DOCKER_BFLASK_IMAGE'
            }
        }
        stage('Test') {
            steps {
                // Introduce a fault in your application code or tests
                // For example, modify a critical file or intentionally fail a test
                sh 'docker run my-flask python -m pytest app/tests/'
            }
        }
        stage('Deploy') {
            steps {
                // Deploy the faulty version
                withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                    sh 'docker push $DOCKER_BFLASK_IMAGE'
                }
            }
        }
        stage('Rollback') {
            steps {
                // Rollback to the previous version
                sh "docker pull $ROLLBACK_IMAGE"
                sh "docker tag $ROLLBACK_IMAGE sayeda77my-flask:previous"
                // You may need additional steps based on your deployment strategy
                // For example, restart the application, clear caches, etc.
            }
        }
    }

    post {
        always {
            // Confirm the rollback by restarting the application with the previous version
            sh 'docker rm -f mypycont'
            sh 'docker run --name mypycont -d -p 3000:5000 my-flask'

            // Notify about the status, you can customize this as needed
            mail to: "aasifamushu@gmail.com",
                 subject: "Rollback Notification from Jenkins",
                 body: "Rollback to the previous version completed successfully."
        }
    }
}
