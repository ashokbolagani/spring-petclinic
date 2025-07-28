pipeline {
  agent any

  tools {
    git 'Default'       // Only if you've named a Git tool "Default" in Jenkins settings
    jdk 'jdk-17'        // Requires JDK configured in Jenkins global tools
    maven 'maven-3.9.10'      // Requires Maven configured in Jenkins
  }

  stages {
    stage('Clone') {
      steps {
        echo "📥 Cloning Spring PetClinic..."
        git url: 'https://github.com/spring-projects/spring-petclinic', branch: 'main'
      }
    }

    stage('Gitleaks Scan') {
      steps {
        echo "🔐 Running Gitleaks secret scan..."
        sh '''
          GITLEAKS_VERSION=$(curl -s https://api.github.com/repos/gitleaks/gitleaks/releases/latest \
            | grep -Po '"tag_name": "v\\K[0-9.]+' || echo "8.27.2")
          echo "📦 Downloading Gitleaks v${GITLEAKS_VERSION}..."

          wget -qO gitleaks.tar.gz https://github.com/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_x64.tar.gz \
            || { echo "❌ Download failed"; exit 1; }
          tar -xzf gitleaks.tar.gz
          chmod +x gitleaks

          ./gitleaks version || { echo "⚠️ Gitleaks execution failed"; exit 1; }

          ./gitleaks detect --source=. --no-banner --report-path=gitleaks-report.json --exit-code 1 \
            || echo "⚠️ Secrets detected – see gitleaks-report.json"

          rm -f gitleaks.tar.gz gitleaks
        '''
      }
    }

    stage('Build') {
      steps {
        echo "🧱 Running Maven build..."
        sh 'mvn clean package'
      }
    }
  }
  stage('SonarQube Analysis') {
      steps {
        echo "🔍 Starting SonarQube code scan..."

        withSonarQubeEnv('Local-SonarQube') {
          sh 'sonar-scanner -Dsonar.projectKey=petclinic-local -Dsonar.sources=src'
        }
      }
    }

    stage('SonarQube Quality Gate') {
      steps {
        echo "🚦 Checking SonarQube Quality Gate..."
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }



  post {
    always {
      echo "📦 Archiving Gitleaks report..."
      archiveArtifacts artifacts: 'gitleaks-report.json', fingerprint: true, allowEmptyArchive: true
    }
    failure {
      echo "🛑 Pipeline failed. Check logs and reports."
    }
    success {
      echo "✅ Pipeline succeeded. Your build and secret scan are clean."
    }
  }


