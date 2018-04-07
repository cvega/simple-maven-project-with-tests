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
        withCredentials([string(credentialsId: 'sonar', variable: 'sonar')]) {
          sh "mvn sonar:sonar -Dsonar.junit.reportsPath=target/surefire-reports -Dtarget/test-classes -Dsonar.host.url=http://sonar.k8s.city -Dsonar.login=${sonar}"
        }
      }
      stage('archive') {
        pom = readMavenPom file: "pom.xml"
        nexusArtifactUploader(
          nexusVersion: "nexus3",
          protocol: "https",
          nexusUrl: "nexus.k8s.city",
          groupId: "demo",
          version: BUILD_NUMBER,
          repository: "maven-releases",
          credentialsId: "nexus",
          artifacts: [
            [
              artifactId: "simple-maven-project-with-tests-jar",
              type: "jar",
              classifier: "debug",
              file: "target/simple-maven-project-with-tests-${pom.version}.jar"
            ]
          ]
        )
        sh "java -jar simple-maven-project-with-tests-jar-${BUILD_NUMBER}-debug.jar"
      }
    }
  }
}

