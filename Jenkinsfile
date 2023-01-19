pipeline {
    agent none
    options {
      timestamp()
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
