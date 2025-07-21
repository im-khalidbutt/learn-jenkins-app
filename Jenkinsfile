pipeline {
    agent any

    stages {
        stage('Hello') {
            agent {
                docker {
                    image 'node:18-apline'
                    resuseNode true
                }
            }
            steps {
               sh '''
                ls-la
                node --version
                npm --version
                npm ci 
                npm run build
                ls-la
 
               ''' 
            }
        }        
    }
}
