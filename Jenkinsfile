pipeline {
  agent any
  options { timestamps() }
  environment { NOTIFY = 's223788901@deakin.edu.au' }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/your_github_username/8.2CDevSecOps.git'
      }
    }
    stage('Install Dependencies') { steps { bat 'npm ci || npm install' } }

    stage('Run Tests') {
      steps {
        script {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            bat 'npm test > test-log.txt 2>&1'
          }
        }
      }
      post {
        always { archiveArtifacts artifacts: 'test-log.txt', fingerprint: true }
        success { emailext to: env.NOTIFY, subject: "TESTS SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                          body: "Tests passed. Build: ${env.BUILD_URL}",
                          attachmentsPattern: 'test-log.txt' }
        failure { emailext to: env.NOTIFY, subject: "TESTS FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                          body: "Tests failed. Build: ${env.BUILD_URL}",
                          attachmentsPattern: 'test-log.txt' }
      }
    }

    stage('Generate Coverage Report') { steps { bat 'npm run coverage || exit /b 0' } }

    stage('NPM Audit (Security Scan)') {
      steps {
        script {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            bat """
              npm audit > audit-log.txt 2>&1
              npm audit --json > audit-results.json 2>&1
            """
          }
        }
      }
      post {
        always { archiveArtifacts artifacts: 'audit-log.txt,audit-results.json', fingerprint: true }
        success { emailext to: env.NOTIFY, subject: "SECURITY SCAN CLEAN: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                          body: "No blocking issues. Build: ${env.BUILD_URL}",
                          attachmentsPattern: 'audit-log.txt,audit-results.json' }
        failure { emailext to: env.NOTIFY, subject: "SECURITY SCAN VULNS FOUND: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                          body: "Vulnerabilities detected. Build: ${env.BUILD_URL}",
                          attachmentsPattern: 'audit-log.txt,audit-results.json' }
      }
    }
  }
}
