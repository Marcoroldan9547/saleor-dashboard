pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }
  environment {
    SONARQUBE_ENV = 'sonarqube'
    SCANNER = 'SonarScanner'
  }
  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Node Install') {
      steps {
        bat """
          if exist package-lock.json ( npm ci ) else ( npm install )
        """
      }
    }

    stage('Tests + Coverage') {
      steps {
        bat "npm run test:ci || npm test -- --ci --coverage --coverageReporters=lcov"
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'junit.xml'
          archiveArtifacts artifacts: 'coverage/lcov.info,junit.xml', fingerprint: true, onlyIfSuccessful: false
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_ENV}") {
          withEnv(["PATH+SCAN=${tool(SCANNER)}\\bin"]) {
            bat 'sonar-scanner.bat -Dsonar.projectKey=saleor-dashboard'
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          script {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') error "Quality Gate: ${qg.status}"
          }
        }
      }
    }

    stage('Build/Deploy (by branch)') {
      when { anyOf { branch 'DEV'; branch 'QA'; branch 'PROD' } }
      steps {
        script {
          echo "Deploy ${env.BRANCH_NAME} (placeholder)"
        }
      }
    }
  }
  post { always { cleanWs() } }
}
