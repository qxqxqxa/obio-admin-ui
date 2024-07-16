pipeline {
    agent any
    stages {
        stage('Build') { 
            steps {
                sh '''
                npm install
                npm run build:prod
                cp -R ./dist/* /var/www/admin
                ''' 
            }
        }
    }
}