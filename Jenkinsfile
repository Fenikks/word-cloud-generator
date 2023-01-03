pipeline {
    agent any
    options {
        timestamps()
    }
    tools{
        go 'go 1.16'
    }
    environment{
        NEXUS_URL='192.168.33.90'  
        STAGE_IP='192.168.33.80'
    }
    stages{

        stage('Get git repo') {
            steps {
                echo "Hello from git"
            }
        }
    }
}
