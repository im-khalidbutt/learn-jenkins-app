pipeline {
    agent any

    stages {
        stage('Hello') {
            agent {
                docker {
                    image 'node:18-apline'
                    reuseNode true
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
