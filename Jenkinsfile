pipeline {
    agent any
    
    environment {
        NODE_ENV = 'production'
        DOCKER_IMAGE = 'monproj'
        DOCKER_REGISTRY = 'my-docker-registry.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'git-credentials-id', url: 'https://github.com/daoudielhassan/monproj.git', branch: 'main'
            }
        }
        
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", 'docker-credentials-id') {
                        docker.image("${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-ssh-server',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'Dockerfile',
                                removePrefix: '',
                                remoteDirectory: '/path/to/remote/deploy',
                                execCommand: 'docker-compose down && docker-compose up -d'
                            )
                        ]
                    )
                ])
            }
        }
    }
    
    post {
        always {
            sh 'docker system prune -f'
        }
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed!'
        }
    }
}

