pipeline {
  agent any
  tools {
    maven 'Maven'
  }
  stages {
    stage('Initialize') {
      steps {
        sh '''

        echo "PATH = ${PATH}"
        echo "M2_HOME = ${M2_HOME}"

        '''
      }
    }
    stage('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/testytest-cicd/JavaApp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
  stage('Source Composition Analysis') {
    steps {
      sh 'rm owasp* || true'
      sh 'wget "https://raw.githubusercontent.com/testytest-cicd/webapp/master/owasp-dependency-check.sh" '
      sh 'chmod +x owasp-dependency-check.sh'
      sh 'bash owasp-dependency-check.sh'
      sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'

    }
  }
  stage('SAST') {
    steps {
      withSonarQubeEnv('sonar') {
        sh 'mvn clean install sonar:sonar'
        sh 'cat target/sonar/report-task.txt'
      }
    }
  }
  stage('Build') {
    steps {
      sh 'mvn clean package'
    }
  }

  stage('Deploy-To-Tomcat') {
    steps {
      sshagent(['tomcat']) {
        sh 'scp -o StrictHostKeyChecking=no uptime target/*.war ubuntu@161.35.71.25:/opt/tomcat/webapps/webapp.war'
      }
    }
  }
  stage('DAST') {
    steps {
      sshagent(['zap']) {
        sh 'ssh -o  StrictHostKeyChecking=no ubuntu@161.35.71.25 uptime "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://161.35.71.25:3000/#/" || true'
      }
    }
  }
}
}

