pipeline {
  agent any


  environment {
    APP_NAME = "owasp-vuln-app"
    DEFAULT_BRANCH = 'master'
    USER_ID = sh(returnStdout: true, script: 'echo \$(git show -s --pretty=%an)').trim()
  }

  stages {

    stage ('Compute version - default branch') {
      when { branch DEFAULT_BRANCH }
      steps {
        script {
          env.APP_VERSION = env.BRANCH_NAME;
        }
      }
    }
    stage("Gradle Build") {
      steps {
        trace('Building ...') {
            sh "./gradlew clean assemble"
        }
      }
    }

    stage ('Dependency track') {
        when { branch DEFAULT_BRANCH }
          steps {
            // First generate Bill of Materials
            sh './gradlew cyclonedxBom -info'
            // Then ingest results to dependency track platform
            withCredentials([string(credentialsId: 'dependency-track-api-key-global', variable: 'API_KEY')])
            {
              dependencyTrackPublisher artifact:'build/reports/bom.xml', projectName:"${env.APP_NAME}", projectVersion:"${env.APP_VERSION}", projectId:"c685e307-defc-4282-b8be-b01c8ee3b5cb", synchronous:true, dependencyTrackApiKey:"${API_KEY}"
             }
          }
    }
  }
}
