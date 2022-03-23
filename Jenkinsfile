pipeline {
    agent {
        label 'master'
    }
     options {
    buildDiscarder(logRotator(numToKeepStr: '7', artifactNumToKeepStr: '7'))
    }
    triggers {
        cron('00 02 * * *')
    }
    environment {
        jenkins_name="aias-ci"
        artifactoryCredentialId="jenkins-aias-artifactory"
        // system environment
        serverId="artifactory_url"
        def BUILDVERSION = sh(script: "echo `date +%Y%m%d`", returnStdout: true).trim()
        def OLDBUILDVERSION = sh(script: "echo `date -d '-7 day' '+%Y%m%d'`", returnStdout: true).trim()
        backup_name="jenkins_backup_${BUILDVERSION}.tar.gz"
        artifactoryRepoPath="tools/jenkins-backups/${jenkins_name}/jenkins_backup_${BUILDVERSION}.tar.gz"
        oldArtifactoryRepoPath="tools/jenkins-backups/${jenkins_name}/jenkins_backup_${OLDBUILDVERSION}.tar.gz"
        

    }
    stages {
        stage('CheckOut'){
            steps{
                git branch: 'master', credentialsId:'scm_user', url:'https://scm_url/git/scm/devops/jenkins-backup.git'
            }
        }
        stage("Start JenkinsBackup") {
        steps{

           
            sh """
             bash scripts/jenkins-backup.sh ${JENKINS_HOME} ${backup_name}
            """
            
        }
        }
        stage("Push Artifactory"){
            steps{
             	rtUpload (
                    serverId: "${serverId}",
                    spec: """{
                            "files": [
                            {
                                "pattern": "${WORKSPACE}/${backup_name}",
                                "target": "${artifactoryRepoPath}",
                                "props" : "isValid=false"
                            }
                            ]
                        }"""
                    )
                
                rtBuildInfo (
                    captureEnv: true
                )
            }
        }
        stage("Delete old Backup Artifactory"){
            steps{
                withCredentials([usernameColonPassword(credentialsId: "${artifactoryCredentialId}", variable: 'USERPASS')]) {
                sh """
                curl -k -u "$USERPASS" -X DELETE https://${serverId}/artifactory/${oldArtifactoryRepoPath}
                """
                }
            }
        }
    }
    post { 
        success { 
            cleanWs()
        }
    }
}
