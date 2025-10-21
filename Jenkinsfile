pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9'
        jdk 'Java17'
    }
    
    environment {
        DOCKER_IMAGE = 'abdessalamzarrouk/projetstage-app:latest '
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
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
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo '🎯 Vérification du Quality Gate...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                echo '🐳 Construction de l\'image Docker...'
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Docker Push') {
            steps {
                echo '📤 Push vers Docker Hub...'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker push ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Deploy') {
            steps {
                echo '🚀 Déploiement de l\'application...'
                sh '''
                    # Arrêter et supprimer l'ancien conteneur
                    docker stop projetstage || true
                    docker rm projetstage || true
                    
                    # Déployer la nouvelle version
                    docker run -d \
                        --name projetstage \
                        --network devops-network \
                        -p 8081:8080 \
                        -e SPRING_DATASOURCE_URL=jdbc:postgresql://my-postgres:5432/postgres \
                        -e SPRING_DATASOURCE_USERNAME=postgres \
                        -e SPRING_DATASOURCE_PASSWORD=abdouzaza \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                    
                    # Attendre le démarrage
                    echo "⏳ Attente du démarrage..."
                    sleep 15
                    
                    # Vérifier que le conteneur tourne
                    docker ps | grep projetstage
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline exécuté avec succès !'
            echo "🐳 Image Docker : ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "🌐 Application déployée sur http://localhost:8081"
        }
        failure {
            echo '❌ Le pipeline a échoué.'
        }
        always {
            echo '🧹 Nettoyage...'
            sh 'docker logout || true'
            cleanWs()
        }
    }
}