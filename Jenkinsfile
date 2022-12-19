library identifier: 'neom-jenkins-sharedlib-terraform@feature/oci-tf-shared-lib', retriever: modernSCM([$class: 'GitSCMSource',
   remote: 'https://github.com/NEOM-KSA/dx-neom-jenkins-sharedlib-terraform.git',
   credentialsId: 'github_token'])

library identifier: 'neom-jenkins-sharedlib-dockerbuild@master', retriever: modernSCM([$class: 'GitSCMSource',
   remote: 'https://github.com/NEOM-KSA/dx-neom-jenkins-sharedlib-dockerbuild.git',
   credentialsId: 'github_token'])

library identifier: 'neom-jenkins-sharedlib-changelog@master', retriever: modernSCM([$class: 'GitSCMSource',
   remote: 'https://github.com/NEOM-KSA/dx-neom-jenkins-sharedlib-changelog.git',
   credentialsId: 'github_token'])

library identifier: 'neom-jenkins-sharedlib-trufflehog_nix@master', retriever: modernSCM([$class: 'GitSCMSource',
   remote: 'https://github.com/NEOM-KSA/dx-neom-jenkins-sharedlib-trufflehog_nix.git',
   credentialsId: 'github_token'])

library identifier: 'neom-jenkins-sharedlib-infra@main', retriever: modernSCM([$class: 'GitSCMSource',
   remote: 'https://github.com/NEOM-KSA/dx-neom-jenkins-sharedlib-infra.git',
   credentialsId: 'github_token'])

library identifier: 'neom-jenkins-sharedlib-teams-notification@master', retriever: modernSCM([$class: 'GitSCMSource',
   remote: 'https://github.com/NEOM-KSA/dx-neom-jenkins-sharedlib-teams-notification.git',
   credentialsId: 'github_token'])

pipeline {
  agent {
    kubernetes {
       yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: gh-cli
            #image: jed.ocir.io/axnfm4jb3i73/mgmt_tools/neom-fss-github-cli-tool:v0.2.0
            image: jed.ocir.io/axnfm4jb3i73/mgmt_tools/neom-fss-github-cli-tool:v0.2.1
            #image: jed.ocir.io/axnfm4jb3i73/github-cli-tool:latest
            command:
            - cat
            tty: true
          - name: trufflehog
            image: jed.ocir.io/axnfm4jb3i73/trufflehog1:latest
            command:
            - cat
            tty: true
          - name: terraform
            image: jed.ocir.io/axnfm4jb3i73/terraform-oci:latest
            command:
            - cat
            tty: true
          - name: terraform-compliance
            image: jed.ocir.io/axnfm4jb3i73/terraform-compliance1:latest
            command:
            - cat
            tty: true
          - name: git
            image: jed.ocir.io/axnfm4jb3i73/git1:latest
            command:
            - cat
            tty: true
          - name: ubuntu
            image: jed.ocir.io/axnfm4jb3i73/ubuntu1:latest
            command:
            - cat
            tty: true
          - name: build
            image: jed.ocir.io/axnfm4jb3i73/gradlejdk11:latest
            command:
            - cat
            tty: true
          - name: changelog
            image: jed.ocir.io/axnfm4jb3i73/changlog1:latest
            command:
            - cat
            tty: true
          imagePullSecrets:
          - name: default-secret
        '''
    }
  }
    environment {
    TENANCY=credentials('tenancy_ocid')
    USER=credentials('user_ocid')
    FINGERPRINT=credentials('fingerprint')
    userpat=credentials('git_token1')
    access_key=credentials('access_key')
    secret_key=credentials('secret_key')	
    def projectName = 'neom-fss-infra-oci-mgmt'
    NAMESPACE = "axnfm4jb3i73"
    BUCKET_NAME = "bucket-android-builds"
    // webhook=credentials('MS_Teams_Webhook_url')
    ENVIRONMENT = "bld"

        // Fastlane Environment Variables
        PATH = "$HOME/.fastlane/bin:" +
                "$HOME/.rvm/gems/ruby-2.5.3/bin:" +
                "$HOME/.rvm/gems/ruby-2.5.3@global/bin:" +
                "$HOME/.rvm/rubies/ruby-2.5.3/bin:" +
                "/usr/local/bin:" +
                "$PATH"
        LC_ALL = "en_US.UTF-8"
        LANG = "en_US.UTF-8"

        VERSION_NAME = ""
        VERSION_SUFFIX = ""
        APP_VERSION_NAME = ""
        VERSION_CODE = ""
        DROPBOX_FOLDER = ""
        PROGUARD_ENABLED = ""
        JIRA_PROJECT_KEY = ""
        PROJECT_NAME = env.JOB_NAME.tokenize("/").first().replaceAll(" Android", "")
    }

    options {
        // Stop the build early in case of compile or test failures
        skipStagesAfterUnstable()
    }

    stages {

    stage('clean workspace') {
      steps {
        cleanWs()
      }
    }

    stage('checkout') {
      steps {
        checkout scm
      }
    }

    stage('oci-authentication') {
      steps {
        container('terraform') {
          script{
            tfci.config()
          }
        }
      }
    }

    stage('Pushing Environment Trigger Details') {
      steps {
        container('terraform') {
          script{
            sh "echo envTriggered: ${envTriggered} > envTriggered"
            sh "oci os object put -ns ${NAMESPACE} -bn ${BUCKET_NAME} --name envTriggered --file envTriggered --force"
          }
        }
      }
    }

    stage("git-authentication-for-tf-mods") {
      steps {
        script {
          container('terraform') {
            sh '''
              git config --global url.https://da00891219:$userpat@github.com/.insteadOf https://github.com/
            '''
          }
        } 
      }
    }
        
        stage('Copy Key Stores') {
            steps {
                script {
                    def projName = PROJECT_NAME.replaceAll(" ", "_").toLowerCase()
                    container('build') {    
                        sh "cp ~/Documents/android-keystores/${projName}_release.jks ../"
                        sh "cp ~/Documents/android-keystores/${projName}_upload.jks ../"
                        sh "cp ~/Documents/android-keystores/debug.ks ../"
                    }
                }
            }
        }

        stage('Copy Local Properties') {
            steps {
                script {
                    def projName = PROJECT_NAME.replaceAll(" ", "_").toLowerCase()
                    container('build') { 
                    sh "cp ~/Documents/${projName}/local.properties ."
                    }
                }
            }
        }

        stage('Setup Versions') {
            steps {
                script {
                    container('build') {
                    VERSION_NAME = sh(
                            script: './gradlew -q printVersionName',
                            returnStdout: true
                    ).trim().tokenize().last()

                    VERSION_SUFFIX = sh(
                            script: './gradlew -q printVersionSuffix',
                            returnStdout: true
                    ).trim().tokenize().last()

                    APP_VERSION_NAME = VERSION_NAME + VERSION_SUFFIX

                    VERSION_CODE = sh(
                            script: './gradlew -q printVersionCode',
                            returnStdout: true
                    ).trim().tokenize().last()

                    JIRA_PROJECT_KEY = sh(
                            script: './gradlew -q printJiraProjectKey',
                            returnStdout: true
                    ).trim().tokenize().last()

                    DROPBOX_FOLDER = "${PROJECT_NAME}/${VERSION_NAME}/${APP_VERSION_NAME}"

                    PROGUARD_ENABLED = sh(
                            script: './gradlew -q printProguardEnabled',
                            returnStdout: true
                    ).trim().tokenize().last()
                    }
                }
            }
        }

        stage('Build App') {
            steps {
                script {
                    container('terraform') {
                    // Clean and assemble APKs
                    sh './gradlew clean assembleDebug assembleRelease'

                    if (env.BRANCH_NAME.startsWith("release")) {
                            sh './gradlew bundleUpload'
                    }
                    }
                }
            }
        }

        // stage('Upload To Google Play Store') {
        //     steps {
        //         script {
        //             if (env.BRANCH_NAME.startsWith("release")) {
        //                 // This Gradle task comes from the Publisher script:
        //                 sh './gradlew generateReleaseMessage uploadToPlayStore tagUploadBuildCommit'

        //                 def repo = scm.getUserRemoteConfigs()[0].getUrl().replaceAll("https://", "")

        //                 withCredentials([usernamePassword(credentialsId: 'Android', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
        //                     def user = URLEncoder.encode(GIT_USERNAME, "UTF-8")
        //                     def pass = URLEncoder.encode(GIT_PASSWORD, "UTF-8")
        //                     repo = "https://${user}:${pass}@${repo}"

        //                     try {
        //                         sh("""
        //                            set +x
        //                            git push ${repo} --tags
        //                            set -x
        //                            """.stripMargin().stripIndent())
        //                     } catch (Exception e) {
        //                         echo e.printStackTrace()
        //                     }
        //                 }
        //             } else {
        //                 echo 'Skipping \'Upload To Google Play Store\' stage since we\'re not on a release branch...'
        //             }
        //         }
        //     }
        // }

        // stage('Archive Files') {
        //     steps {
        //         // Archive the APKs so that they can be downloaded from Jenkins
        //         echo 'Archiving APKs...'
        //         archiveArtifacts '**/*.apk'

        //         script {
        //             if (PROGUARD_ENABLED == "true") {
        //                 echo 'Archiving Mappings...'
        //                 archiveArtifacts '**/mapping.txt'
        //             }

        //             if (env.BRANCH_NAME.startsWith("release")) {
        //                 echo 'Archiving AABs...'
        //                 archiveArtifacts '**/*.aab'
        //             }
        //         }
        //     }
        // }

        // stage('Upload to Dropbox') {
        //     steps {
        //         script {
        //             if (env.BRANCH_NAME.startsWith("release")) {
        //                 dropbox configName: 'Dropbox Android', remoteDirectory: "${DROPBOX_FOLDER}", sourceFiles: '**/*.apk', flatten: true
        //             } else {
        //                 echo 'Skipping \'Upload to Dropbox\' stage since we\'re not on a release branch...'
        //             }
        //         }
        //     }
        // }

    stage('Publish Artifact to OCI Storage Bucket') {
      steps {
        container('terraform') {
          script{
            try {
              sh "oci os object bulk-upload -ns ${NAMESPACE} --bucket-name ${BUCKET_NAME} --src-dir ${WORKSPACE}/*.apk --object-prefix apk_files/ --overwrite"
            }
            catch (Exception e) {
              error("${e}")
            }
          }
        }
      }
    }

        stage('Static Analysis') {
            steps {
                echo 'Static Analysis'
                // Run Lint and analyse the results
                container('terraform') {
                sh './gradlew lintProductionRelease'
                androidLint()
                }
            }
        }

        stage('Finished') {
            steps {
                echo 'Finished'
            }
        }
    }

    // post {
    //     success {
    //         script {
    //             bitbucketStatusNotify(buildState: 'SUCCESSFUL')

    //             // Notify if the upload succeeded
    //             if (env.BRANCH_NAME.startsWith("release")) {

    //                 def dropboxFolderLink = "https://www.dropbox.com/home/Android/${DROPBOX_FOLDER}"
    //                 dropboxFolderLink = dropboxFolderLink.replaceAll(" ", "%20")

    //                 currentBuild.displayName = "${currentBuild.displayName}-${APP_VERSION_NAME}-${VERSION_CODE}"

    //                 mail to: "${QA_MAILS}",
    //                         cc: "${MANAGMENT_MAILS}, ${DEV_MAILS}",
    //                         subject: "${PROJECT_NAME} Android - New Build ${APP_VERSION_NAME}",
    //                         body: """\
    //                 ${PROJECT_NAME} Android - Build ${APP_VERSION_NAME} is available for testing.

    //                 The build is uploaded in Google Play Store Internal Test track.

    //                 Also available in:
    //                     Dropbox at - ${dropboxFolderLink}
    //                     Jenkins at - ${env.BUILD_URL}

    //                 Resolved issues and tasks - https://jira.example.com/issues/?jql=project%20%3D%20${JIRA_PROJECT_KEY}%20AND%20fixVersion%20%3D%20${APP_VERSION_NAME}

    //                 ---------------------------------
    //                 This is an automatic message, please do not reply!\
    //                 """.stripMargin().stripIndent()
    //             }
    //         }
    //     }
    //     failure {
    //         script {
    //             bitbucketStatusNotify(buildState: 'FAILED')

    //             // Notify developer team of the failure
    //             mail to: "${DEV_MAILS}",
    //                     subject: "${env.JOB_NAME} - Build ${APP_VERSION_NAME} #${env.BUILD_NUMBER} - FAILED!",
    //                     body: """\
    //             ${env.JOB_NAME} - Build ${APP_VERSION_NAME} #${env.BUILD_NUMBER} - FAILED!:

    //             Check it out at ${env.BUILD_URL}

    //             ---------------------------------
    //             This is an automatic message, please do not reply!\
    //             """.stripMargin().stripIndent()
    //         }
    //     }
    // }
}