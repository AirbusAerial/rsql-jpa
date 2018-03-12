def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')

podTemplate(label: label, containers: [
    containerTemplate(name: 'maven', image: 'maven:3-jdk-8-alpine', ttyEnabled: true, command: 'cat',
        envVars: [
            envVar(key: 'SPRING_PROFILES_ACTIVE', value: 'ci'),
            secretEnvVar(key: 'CI_MVN_USERNAME', secretName: 'nexus-credentials', secretKey: 'username'),
            secretEnvVar(key: 'CI_MVN_PASSWORD', secretName: 'nexus-credentials', secretKey: 'password')]
    )],
    volumes: [
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        hostPathVolume(hostPath: '/var/jenkins_home/repo', mountPath: '/root/.m2/repository'),
        secretVolume(secretName: 'instance-credentials', mountPath: '/secrets/google')]) {

    if  (env.BRANCH_NAME == 'master') {
        node(label) {
            try {
                checkout scm

                stage('Build') {
                    container('maven') {
                        sh 'mvn deploy -B -s settings.xml'
                    }
                    notifyBuild('PUBLISHED')
                }
            } catch (e) {
                currentBuild.result = "FAILURE"
                throw e
            } finally {
                if (currentBuild.result == 'FAILURE') {
                    notifyBuild(currentBuild.result)
                }
            }
        }
    }

}

def notifyBuild(String buildStatus) {
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  if (buildStatus == 'FAILURE') {
    color = 'RED'
    colorCode = '#FF0000'
  } else {
    color = 'GREEN'
    colorCode = '#00FF00'
  }

  slackSend (color: colorCode, message: summary)
}