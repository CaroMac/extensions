def mvnProfile    = 'dev'
def galasaSignJarSkip = 'true'

pipeline {
// Initially run on any agent
   agent {
      label 'codesigning'
   }
   environment {
//Configure Maven from the maven tooling in Jenkins
      def mvnHome = tool 'Default'
      PATH = "${mvnHome}/bin:${env.PATH}"

//Set some defaults
      def workspace = pwd()
      def mvnGoal    = 'install'
   }
   stages {
// If it is the master branch, version 0.3.0 and master on all the other branches
      stage('set-dev') {
         when {
           environment name: 'GIT_BRANCH', value: 'origin/master'
         }
         steps {
            script {
               mvnProfile        = 'dev'
               mvnGoal           = 'deploy sonar:sonar'
               galasaSignJarSkip = 'false'
            }
         }
      }
// If it is the staging branch
      stage('set-staging') {
         when {
           environment name: 'GIT_BRANCH', value: 'origin/staging'
         }
         steps {
            script {
               mvnGoal           = 'deploy'
               mvnProfile        = 'staging'
               galasaSignJarSkip = 'false'
            }
         }
      }

// for debugging purposes
      stage('report') {
         steps {
            echo "Branch/Tag         : ${env.GIT_BRANCH}"
            echo "Workspace directory: ${workspace}"
            echo "Maven Goal         : ${mvnGoal}"
            echo "Maven profile      : ${mvnProfile}"
            echo "Skip Signing JARs  : ${galasaSignJarSkip}"
         }
      }

// Set up the workspace, clear the git directories and setup the manve settings.xml files
      stage('prep-workspace') {
         steps {
            configFileProvider([configFile(fileId: '86dde059-684b-4300-b595-64e83c2dd217', targetLocation: 'settings.xml')]) {
            }
            dir('repository/dev.galasa') {
               deleteDir()
            }
            dir('repository/dev/galasa') {
               deleteDir()
            }
         }
      }

      stage('Extensions Maven') {
         steps {
            withCredentials([string(credentialsId: 'galasa-gpg', variable: 'GPG')]) {
               withSonarQubeEnv('GalasaSonarQube') {
                  dir('galasa-extensions-parent') {
                     sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"

                     dir('dev.galasa.cps.etcd') {
                        sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Djarsigner.skip=${galasaSignJarSkip} -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"
                     }

                     dir('dev.galasa.ras.couchdb') {
                       sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Djarsigner.skip=${galasaSignJarSkip} -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"
                     }

                     dir('dev.galasa.extensions.obr') {
                       sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"
                     }

                     dir('dev.galasa.devtools.karaf') {
                        sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"
                     }

                     dir('dev.galasa.jenkins') {
                        sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -DskipTests -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"
                     }
                  }
               }
            }
         }
      }
      stage('Eclipse Maven') {
         steps {
            withCredentials([string(credentialsId: 'galasa-gpg', variable: 'GPG')]) {
               withSonarQubeEnv('GalasaSonarQube') {
                  dir('galasa-eclipse-parent') {
                     sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"

                     dir('dev.galasa.eclipse') {
                        sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Djarsigner.skip=${galasaSignJarSkip} -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"
                     }

                     dir('dev.galasa.eclipse.feature') {
                        sh "mvn --settings ${workspace}/settings.xml -Dmaven.repo.local=${workspace}/repository -Djarsigner.skip=${galasaSignJarSkip} -Dgpg.skip=false -Dgpg.passphrase=$GPG -P ${mvnProfile} -B -e -fae --non-recursive ${mvnGoal}"
                     }
                  }
               }
            }
         }
      }
   }
   post {
       // triggered when red sign
       failure {
           slackSend (channel: '#project-galasa-devs', color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
       }
    }
}
