 pipeline {
    agent {
        docker {
            // Sử dụng image Maven với Java 17
            image 'maven:3.8.6-openjdk-17-slim'
            args '-v /root/.m2:/root/.m2 -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        DOCKER_REGISTRY_CREDENTIAL = 'docker-registry-credentials'
        APP_NAME = 'taskapi'
        VERSION = "${env.BUILD_ID}"
        // Thêm biến môi trường cho GitHub
        GITHUB_REPO = 'https://github.com/Huytulato/taskapi-.git'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout từ GitHub repository
                checkout scm
                // Hiển thị thông tin phiên bản Java và Maven
                sh 'java -version'
                sh 'mvn -version'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        
        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker version'
                    
                    // Thêm tạo Dockerfile nếu chưa có trong repo
                    sh '''
                    if [ ! -f Dockerfile ]; then
                        cat > Dockerfile << EOF
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
EOF
                    fi
                    '''
                    
                    sh "docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${VERSION} ."
                    sh "docker tag ${DOCKER_REGISTRY}/${APP_NAME}:${VERSION} ${DOCKER_REGISTRY}/${APP_NAME}:latest"
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: DOCKER_REGISTRY_CREDENTIAL, variable: 'DOCKER_PASSWORD')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login ${DOCKER_REGISTRY} -u admin --password-stdin"
                    }
                    
                    sh "docker push ${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}"
                    sh "docker push ${DOCKER_REGISTRY}/${APP_NAME}:latest"
                }
            }
        }
        
        stage('Deploy to Development') {
            steps {
                sh """
                    docker stop ${APP_NAME}-dev || true
                    docker rm ${APP_NAME}-dev || true
                    
                    docker run -d -p 8080:8080 \
                        --name ${APP_NAME}-dev \
                        -e SPRING_PROFILES_ACTIVE=dev \
                        ${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}
                """
                
                sh 'sleep 15'
                sh 'curl -s http://localhost:8080/actuator/health || echo "Health check failed but continuing"'
            }
        }
    }
    
    post {
        success {
            echo 'Build and deployment successful!'
            // Cập nhật trạng thái build thành công trên GitHub
            githubStatus status: 'SUCCESS', context: 'continuous-integration/jenkins', description: 'Build succeeded!'
        }
        failure {
            echo 'Build or deployment failed!'
            // Cập nhật trạng thái build thất bại trên GitHub
            githubStatus status: 'FAILURE', context: 'continuous-integration/jenkins', description: 'Build failed!'
        }
        always {
            cleanWs()
            sh 'docker system prune -f || true'
        }
    }
}
