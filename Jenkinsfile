pipeline {
    agent any

    environment {
        DOCKER_USER        = 'harish230504'
        TAG                = "v${env.BUILD_NUMBER}"
        JAVA_TOOL_OPTIONS  = '-Duser.timezone=Asia/Kolkata'
        DB_URL             = 'jdbc:postgresql://localhost:5432/incidentdb'
        DB_USERNAME        = 'postgres'
        DB_PASSWORD        = 'postgres'
    }

    stages {
        stage('Build') {
            steps {
                dir('backend')  { bat 'mvn -B clean package -DskipTests' }
                dir('frontend') {
                    bat 'npm ci'
                    bat 'npm run build'
                }
            }
        }
        stage('Test') {
            steps {
                dir('backend') { bat 'mvn -B test' }
            }
        }
        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_USER%/employee-backend:%TAG% backend'
                bat 'docker build -t %DOCKER_USER%/employee-frontend:%TAG% --build-arg VITE_API_URL=http://localhost:8088 frontend'
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    bat 'echo %DH_PASS%| docker login -u %DH_USER% --password-stdin'
                    bat 'docker push %DOCKER_USER%/employee-backend:%TAG%'
                    bat 'docker push %DOCKER_USER%/employee-frontend:%TAG%'
                }
            }
        }
        stage('Deploy') {
            steps {
                bat 'kubectl set image deployment/employee-backend employee-backend=%DOCKER_USER%/employee-backend:%TAG%'
                bat 'kubectl set image deployment/employee-frontend employee-frontend=%DOCKER_USER%/employee-frontend:%TAG%'
                bat 'kubectl rollout status deployment/employee-backend --timeout=300s'
                bat 'kubectl rollout status deployment/employee-frontend --timeout=120s'
            }
        }
    }
}
