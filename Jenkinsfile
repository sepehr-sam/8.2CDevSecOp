// Jenkinsfile for Part 2 (Windows agent, no email)
// Repo: https://github.com/sepehr-sam/8.2CDevSecOps.git

pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        // Change branch if your default is 'master'
        git branch: 'main', url: 'https://github.com/sepehr-sam/8.2CDevSecOp.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        // Use reproducible install if lockfile exists; fall back otherwise
        bat 'npm ci || npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // Capture output to a file; don't kill the whole pipeline if tests fail
        script {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            bat 'npm test > test-log.txt 2>&1'
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'test-log.txt', fingerprint: true
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // If no coverage script, keep going quietly
        bat 'npm run coverage || exit /b 0'
      }
      post {
        always {
          archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true
        }
      }
    }

    stage('NPM Audit (Security Scan))') {
      steps {
        // Human-readable + JSON reports; keep pipeline alive even if vulns found
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
        always {
          archiveArtifacts artifacts: 'audit-log.txt,audit-results.json', fingerprint: true
        }
      }
    }
  }
}
