pipeline{
    agent any

    tools{
        maven 'MVN387'
    }

    stages{
        stage('Install Dependencies'){
            steps{
                sh '''
                
                mvn clean install -DskipTests

                '''
            }
        }
    }
}