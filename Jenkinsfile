pipeline {
    agent none
    options {
      timestamps()
    }
    stages {
        stage('Test') {
            agent {
                dockerfile {
                    filename 'Dockerfile'
                }
            } 
           steps {
               sh 'curl http://google.com | wc -c >google-size'
               sh 'cat google-size' 
           }
        }
    }
}
