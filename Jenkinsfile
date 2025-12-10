pipeline {
    agent any
    
    stages {
        stage('Hello') {
            steps {
                echo 'Bonjour depuis mon Jenkinsfile!'
                sh 'echo "Workspace: $WORKSPACE"'
            }
        }
        stage('List Files') {
            steps {
                sh '''
                    echo "Fichiers présents:"
                    ls -la
                    echo "Date: $(date)"
                '''
            }
        }
    }
}
