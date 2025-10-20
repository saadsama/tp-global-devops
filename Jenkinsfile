pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9'
        jdk 'Java17'
    }
    
    environment {
        DOCKER_IMAGE = 'saadsama/tp-global-devops'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📥 Clonage du repository...'
                git branch: 'feature/jenkins-pipeline', 
                    url: 'https://github.com/saadsama/tp-global-devops.git'
            }
        }
        
        stage('Build') {
            steps {
                echo '🔨 Compilation du projet...'
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Exécution des tests unitaires avec H2...'
                sh 'mvn test -Dspring.profiles.active=test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
                
        stage('Package') {
            steps {
                echo '📦 Génération du package...'
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Analyse SonarQube...'
                echo 'SonarQube sera configuré à l\'étape 3'
                // On activera cette étape plus tard
            }
        }
        
        stage('Docker Build') {
            steps {
                echo '🐳 Construction de l\'image Docker...'
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline exécuté avec succès !'
        }
        failure {
            echo '❌ Le pipeline a échoué.'
        }
        always {
            echo '🧹 Nettoyage...'
            cleanWs()
        }
    }
}