pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-cred')
        DOCKER_IMAGE = 'oussama433/mon-jenkins-test'
        VERSION = "${BUILD_NUMBER}"
    }
    
    stages {
        // Étape 1: Checkout GitHub
        stage('Checkout from GitHub') {
            steps {
                echo '📥 Récupération du code depuis GitHub...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/oussama433/mon-jenkins-test.git',
                        credentialsId: 'github-cred'
                    ]]
                ])
                sh 'ls -la'
            }
        }
        
        // Étape 2: Build Maven
        stage('Build with Maven') {
            steps {
                echo '🔨 Construction du projet avec Maven...'
                sh './mvnw clean package -DskipTests'
                sh 'ls -la target/'
            }
        }
        
        // Étape 3: SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Analyse de qualité du code avec SonarQube...'
                script {
                    // Décommentez quand SonarQube est configuré
                    // withSonarQubeEnv('SonarQube-Server') {
                    //     sh './mvnw sonar:sonar \
                    //         -Dsonar.projectKey=mon-jenkins-test \
                    //         -Dsonar.projectName="Mon Jenkins Test"'
                    // }
                    echo '✅ Étape SonarQube simulée (à configurer)'
                    sh 'echo "SonarQube report would be generated here" > sonar-report.txt'
                }
            }
        }
        
        // Étape 4: Build Docker Image
        stage('Build Docker Image') {
            steps {
                script {
                    echo '🐳 Construction de l\'image Docker...'
                    sh 'docker version'
                    dockerImage = docker.build("${DOCKER_IMAGE}:${VERSION}")
                }
            }
        }
        
        // Étape 5: Test Docker Image
        stage('Test Docker Image') {
            steps {
                script {
                    echo '🧪 Test de l\'image Docker...'
                    sh """
                        docker run -d --name test-container ${DOCKER_IMAGE}:${VERSION}
                        sleep 10
                        docker ps
                        docker logs test-container
                        docker stop test-container
                        docker rm test-container
                    """
                }
            }
        }
        
        // Étape 6: Push to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo '📤 Pushing image to Docker Hub...'
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-cred') {
                        dockerImage.push("${VERSION}")
                        dockerImage.push("latest")
                    }
                    echo '✅ Image poussée avec succès!'
                }
            }
        }
        
        // Étape 7: Deploy to Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo '☸️ Déploiement sur Kubernetes...'
                    sh '''
                        echo "=== Déploiement Kubernetes ==="
                        echo "1. Application: ${DOCKER_IMAGE}:${VERSION}"
                        echo "2. Pour déployer manuellement:"
                        echo "   kubectl apply -f deployment-springboot.yaml"
                        echo "   kubectl get pods"
                        echo "   minikube service springboot-service"
                    '''
                    // Pour un vrai déploiement, décommentez:
                    // sh 'kubectl apply -f deployment-springboot.yaml'
                    // sh 'kubectl rollout status deployment/springboot-deployment'
                }
            }
        }
    }
    
    post {
        success {
            echo '🎉 Pipeline exécuté avec succès!'
            sh '''
                echo "========================================"
                echo "✅ PIPELINE RÉUSSI"
                echo "========================================"
                echo "📦 Image Docker: ${DOCKER_IMAGE}:${VERSION}"
                echo "🔗 Docker Hub: https://hub.docker.com/r/oussama433/mon-jenkins-test"
                echo "📊 Build: ${BUILD_NUMBER}"
                echo "🕐 Durée: ${currentBuild.durationString}"
                echo "========================================"
            '''
            // Optionnel: Archive les artifacts
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        failure {
            echo '❌ Pipeline échoué!'
            sh '''
                echo "========================================"
                echo "❌ PIPELINE ÉCHOUÉ"
                echo "========================================"
                echo "Build: ${BUILD_NUMBER}"
                echo "Raison: Vérifiez les logs Jenkins"
                echo "========================================"
            '''
        }
        always {
            // Nettoyage
            sh 'docker system prune -f || true'
            echo '🧹 Nettoyage Docker effectué'
        }
    }
}
