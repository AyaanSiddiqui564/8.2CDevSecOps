pipeline {
  agent any

  environment {
    REPO = 'https://github.com/AyaanSiddiqui564/8.2CDevSecOps.git'
    NOTIFY_EMAIL = 's224777251@deakin.edu.au'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: "${REPO}"
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // run tests and tee output into a stage-specific log file
        sh '''
          npm test || true | tee test-stage.log
        '''
      }
      post {
        always {
          // attach the test-stage.log file created above
          emailext (
            to: "${NOTIFY_EMAIL}",
            subject: "Jenkins: Tests stage - Build #${BUILD_NUMBER} - ${currentBuild.currentResult}",
            body: "Tests stage finished. See attached test-stage.log for details.",
            attachmentsPattern: 'test-stage.log'
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        sh 'npm run coverage || true | tee coverage-stage.log'
        archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // produce audit output and save
        sh '''
          npm audit --json > npm-audit.json || true
          npm audit || true | tee npm-audit.log
        '''
      }
      post {
        always {
          // attach the npm-audit.log to the email
          emailext (
            to: "${NOTIFY_EMAIL}",
            subject: "Jenkins: Security scan - Build #${BUILD_NUMBER} - ${currentBuild.currentResult}",
            body: "Security scan finished. See attached npm-audit.log for details.",
            attachmentsPattern: 'npm-audit.log'
          )
        }
      }
    }

    stage('(Optional) Deploy to Staging') {
      when {
        expression { false } // optionally enable if you have a staging host
      }
      steps {
        echo 'Deploy step placeholder'
      }
    }

    stage('(Optional) Deploy to Production') {
      when {
        expression { false } // optionally enable only when ready
      }
      steps {
        echo 'Deploy to production placeholder'
      }
    }
  }

  post {
    success {
      emailext (
        to: "${NOTIFY_EMAIL}",
        subject: "Jenkins Build #${BUILD_NUMBER} - SUCCESS",
        body: """<p>Good news — the pipeline completed successfully.</p>
                 <p>Project: 8.2CDevSecOps</p>
                 <p>Build: ${BUILD_NUMBER}</p>""",
        attachLog: true
      )
    }

    failure {
      emailext (
        to: "${NOTIFY_EMAIL}",
        subject: "Jenkins Build #${BUILD_NUMBER} - FAILED",
        body: """<p>Pipeline failed. Please review the attached console log for details.</p>
                 <p>Project: 8.2CDevSecOps</p>
                 <p>Build: ${BUILD_NUMBER}</p>""",
        attachLog: true
      )
    }

    always {
      echo "Pipeline finished. Stage notifications were attempted."
    }
  }
}
