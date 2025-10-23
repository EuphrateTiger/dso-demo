pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }

  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container(name: 'maven') {
              sh 'mvn compile'
            }

          }
        }

      }
    }

    stage('SCA') {
        steps {
            container('maven') {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'mvn org.owasp:dependency-check-maven:check'
                }
            }
        }
        post {
            always {
                archiveArtifacts(
                    allowEmptyArchive: true,
                    artifacts: 'target/dependency-check-report.html',
                    fingerprint: true,
                    onlyIfSuccessful: true
                )
                // Pour dependencyCheckPublisher si besoin :
                // dependencyCheckPublisher pattern: 'report.xml'
            }
        }
    }

    stage('Package') {
        parallel {
            stage('Create Jarfile') {
                steps {
                    container(name: 'maven') {
                        sh 'mvn package -DskipTests'
                    }
                }
            }
            stage('OCIImageBnP') {
                steps {
                    container('kaniko') {
                        sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/samuellabrosse/dso-demo'
                    }
                }
            }
        }
    }


    stage('Deploy to Dev') {
      steps {
        sh 'echo done'
      }
    }

  }
}
