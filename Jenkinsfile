pipeline {
  agent any
  triggers { pollSCM('H/2 * * * *') }

  stages {
    stage('Build') {
      steps {
        sh 'tar -czf webapp-latest.tar.gz index.html'
      }
    }

    stage('Upload to Artifactory') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'artifactory-creds',
            usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
          sh '''
            curl -f -u "$ART_USER:$ART_PASS" -T webapp-latest.tar.gz \
              http://localhost:8082/artifactory/webapp-releases/webapp-latest.tar.gz
            cp webapp-latest.tar.gz webapp-${BUILD_NUMBER}.tar.gz
            curl -f -u "$ART_USER:$ART_PASS" -T webapp-${BUILD_NUMBER}.tar.gz \
              http://localhost:8082/artifactory/webapp-releases/webapp-${BUILD_NUMBER}.tar.gz
          '''
        }
      }
    }

    stage('Deploy with Ansible') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'artifactory-creds',
            usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
          sh '''
            ansible-playbook -i inventory.ini deploy.yml \
              -e "artifactory_user=$ART_USER" -e "artifactory_pass=$ART_PASS"
          '''
        }
      }
    }
  }
}
