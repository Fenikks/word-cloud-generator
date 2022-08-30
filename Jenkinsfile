pipeline {
    agent any
    options {
        timestamps()
    }
    environment{
        NEXUS_IP = "192.168.33.90"
        STAGING_IP = "192.168.33.80"
    }
    tools {
        go 'Go 1.16'
    }
    
    stages{
        stage('Get source code') {
            agent {
                label 'go'
            } 
            steps {
                git 'https://github.com/Fenikks/word-cloud-generator.git'

                sh '''  export GOPATH=$WORKSPACE/go
                        export PATH="$PATH:$(go env GOPATH)/bin"
                        go get github.com/tools/godep
                        go get github.com/smartystreets/goconvey
                        go get github.com/GeertJohan/go.rice/rice
                        go get github.com/wickett/word-cloud-generator/wordyapi
                        go get github.com/gorilla/mux
                        sed -i "s/1.DEVELOPMENT/1.${BUILD_NUMBER}/g" static/version
                        GOOS=linux GOARCH=amd64 go build -o ./artifacts/word-cloud-generator -v
                        
                        md5sum artifacts/word-cloud-generator
                        gzip -f artifacts/word-cloud-generator
                        
                        ls -l artifacts'''

                nexusArtifactUploader(
                    artifacts: [
                        [artifactId: 'word-cloud-generator', 
                         classifier: '', 
                         file: 'artifacts/word-cloud-generator.gz', 
                         type: 'gz']
                    ], 
                    credentialsId: 'nexus-uploader', 
                    groupId: "${BRANCH}_jenkinsfile", 
                    nexusUrl: "${NEXUS_IP}:8081/", 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'word-cloud-build', 
                    version: '1.${BUILD_NUMBER}'
                )
            }
        }
        stage('Test install'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'slave', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD' )]){
                    withCredentials([usernamePassword(credentialsId: 'nexus-uploader', usernameVariable: 'USERNAME_N', passwordVariable: 'PASSWORD_N' )]){                    
                        sh '''   sshpass -p $PASSWORD ssh -o StrictHostKeyChecking=no $USERNAME@$STAGING_IP "sudo service wordcloud stop" 
                            sshpass -p $PASSWORD ssh -o StrictHostKeyChecking=no $USERNAME@$STAGING_IP "curl -X GET -u $USERNAME_N:$PASSWORD_N http://192.168.33.90:8081/repository/word-cloud-build/${BRANCH}_jenkinsfile/word-cloud-generator/1.$BUILD_NUMBER/word-cloud-generator-1.$BUILD_NUMBER.gz -o /opt/wordcloud/word-cloud-generator.gz"
                            sshpass -p $PASSWORD ssh -o StrictHostKeyChecking=no $USERNAME@$STAGING_IP "gunzip -f /opt/wordcloud/word-cloud-generator.gz"
                            sshpass -p $PASSWORD ssh -o StrictHostKeyChecking=no $USERNAME@$STAGING_IP "chmod +x /opt/wordcloud/word-cloud-generator"
                            sshpass -p $PASSWORD ssh -o StrictHostKeyChecking=no $USERNAME@$STAGING_IP "sudo service wordcloud start"
                        '''
                    }
                }
            }
        }
    }
}
