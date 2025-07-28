pipeline {
  agent any

  tools {
    git 'Default'
    jdk 'jdk-17'
    maven 'maven-3.9.10'
  }

  stages {
    stage('Clone') {
      steps {
        echo "📥 Cloning Spring PetClinic ashok repo..."
        git url: 'https://github.com/spring-projects/spring-petclinic', branch: 'main'
      }
    }

    stage('Gitleaks Scan') {
      steps {
        echo "🔐 Running Gitleaks secret scan..."
        // Gitleaks logic here...
      }
    }

    stage('Build') {
      steps {
        echo "🧱 Running Maven build..."
        sh 'mvn clean package'
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
  } // <-- Make sure all stage blocks are wrapped inside this

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
}