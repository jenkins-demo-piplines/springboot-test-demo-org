pipeline{
    agent any

    tools{
        maven 'MVN387'
    }

    stages{
        stage('maven Version'){
            steps{
                sh '''
                
                mvn --version

                '''
            }
        }
    }
}