pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/sepehr-sam/8.2CDevSecOp.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        bat "npm install"
      }
    }

    stage('Run Tests') {
      steps {
        bat "npm test || exit /b 0"  // Allows pipeline to continue despite test failures
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // Ensure coverage report exists
        bat "npm run coverage || exit /b 0"
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        bat "npm audit || exit /b 0" // This will show known CVEs in the output
      }
    }
  }
}
