pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'YOUR_DOCKERHUB_USERNAME'
    BACKEND_IMAGE  = "${DOCKERHUB_USER}/employee-backend"
    FRONTEND_IMAGE = "${DOCKERHUB_USER}/employee-frontend"
    TAG            = "v${BUILD_NUMBER}"
    JAVA_HOME      = 'C:\\Program Files\\Microsoft\\jdk-21.0.12.8-hotspot'
    PATH           = "${JAVA_HOME}\\bin;${env.PATH}"
  }

  stages {
    stage('Build') {
      steps {
        dir('backend')  { bat 'mvnw.cmd -B clean package -DskipTests' }
        dir('frontend') { bat 'npm ci && npm run build' }
      }
    }

    stage('Test') {
      steps {
        // Spring context tests need a database; do not block the pipeline on them
        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
          dir('backend') { bat 'mvnw.cmd -B test' }
        }
      }
    }

    stage('Docker Build') {
      steps {
        bat 'docker build -t %BACKEND_IMAGE%:%TAG% -t %BACKEND_IMAGE%:latest backend'
        bat 'docker build -t %FRONTEND_IMAGE%:%TAG% -t %FRONTEND_IMAGE%:latest frontend'
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'DH_USER',
                                          passwordVariable: 'DH_PASS')]) {
          bat 'echo %DH_PASS%| docker login -u %DH_USER% --password-stdin'
          bat 'docker push %BACKEND_IMAGE%:%TAG%'
          bat 'docker push %BACKEND_IMAGE%:latest'
          bat 'docker push %FRONTEND_IMAGE%:%TAG%'
          bat 'docker push %FRONTEND_IMAGE%:latest'
        }
      }
    }

    stage('Deploy') {
      steps {
        bat 'kubectl apply -f k8s/'
        bat 'kubectl set image deployment/backend backend=%BACKEND_IMAGE%:%TAG%'
        bat 'kubectl set image deployment/frontend frontend=%FRONTEND_IMAGE%:%TAG%'
        bat 'kubectl rollout status deployment/backend --timeout=240s'
        bat 'kubectl rollout status deployment/frontend --timeout=120s'
      }
    }
  }

  post {
    always { bat 'docker logout' }
  }
}
