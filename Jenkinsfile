pipeline {
    agent any
    options {
        timestamps()
        
    }
    environment{
        NEXUS_IP='192.168.33.90'
        STAGING_IP='192.168.33.80'
    }
    tools {
        go 'Go-1.16'
    }
    stages{
        stage("get source code"){
            steps{
                git 'https://github.com/Fenikks/word-cloud-generator.git'
            }
        }
        stage('build code') {
            steps{
                sh '''
                    sed -i "s/1.DEVELOPMENT/1.$BUILD_NUMBER/g" static/version
                    
                    GOOS=linux GOARCH=amd64 go build -o ./artifacts/word-cloud-generator -v 
                    
                    md5sum artifacts/word-cloud-generator
                    
                    gzip -f artifacts/word-cloud-generator
                    ls -l artifacts/
                '''
            
            }
        }
        stage('upload to Nexus'){
            steps{
                nexusArtifactUploader (
                    artifacts: [[
                        artifactId: 'word-cloud-generator', 
                        classifier: '', 
                        file: 'artifacts/word-cloud-generator.gz', 
                        type: 'gz']], 
                    credentialsId: 'nexus_uploader', 
                    groupId: 'pipeline', 
                    nexusUrl: "$NEXUS_IP:8081", 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'word-cloud-build', 
                    version: '1.$BUILD_NUMBER'
                )
            }
        }
        
        stage('install on staging'){
            environment{
                NEXUS_CREDS = credentials('nexus_uploader')
            }
            steps{
                    sh '''
                        ssh vagrant@$STAGING_IP "sudo systemctl stop wordcloud"
                        ssh vagrant@$STAGING_IP "curl -X GET "http://$NEXUS_IP:8081/repository/word-cloud-build/pipeline/word-cloud-generator/1.$BUILD_NUMBER/word-cloud-generator-1.$BUILD_NUMBER.gz" -o /opt/wordcloud/word-cloud-generator.gz -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW"
                        ssh vagrant@$STAGING_IP "ls -l /opt/wordcloud"
                        ssh vagrant@$STAGING_IP "gunzip -f /opt/wordcloud/word-cloud-generator.gz"
                        ssh vagrant@$STAGING_IP "chmod +x /opt/wordcloud/word-cloud-generator"
                        ssh vagrant@$STAGING_IP "sudo service wordcloud start"
                    '''

            }
        }
    }
}
