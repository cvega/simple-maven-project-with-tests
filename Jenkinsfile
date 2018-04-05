podTemplate(label: 'kubernetes',
  containers: [
    containerTemplate(name: 'maven', image: 'maven:3.5.2-jdk-8-alpine', ttyEnabled: true, command: 'cat')
  ]) {
  node("kubernetes") {
    container("maven") {
      stage('checkout') {
        git 'https://github.com/cvega/simple-maven-project-with-tests.git';
      }
      stage('build') {
        sh "mvn -Dmaven.test.failure.ignore clean package"
      }
      stage('test') {
        // junit '**/target/surefire-reports/TEST-*.xml'
        withCredentials([string(credentialsId: 'sonar', variable: 'sonar')]) {
          sh "mvn sonar:sonar -Dsonar.host.url=http://sonar.k8s.city -Dsonar.login=${sonar}"
        }
      }
      stage('archive') {
        // archive 'target/*.jar'
        withCredentials([string(credentialsId: 'nexus', variable: 'nexus')]) {
          nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'https',
            nexusUrl: 'nexus.k8s.city',
            version: '3.9.0',
            repository: 'simple-maven-project',
            credentialsId: "${nexus}",
            artifact: [
              artifactId: 'simple-maven-project-uploader',
              type: 'jar',
              classifier: 'debug',
              file: 'target/*.jar'
            ]
          )
        }
      }
    }
  }
}
