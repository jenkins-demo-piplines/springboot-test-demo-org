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

        stage('OWASP Dependency Check'){
            steps{
                dependencyCheck additionalArguments: '''
                --scan \' ./\'
                 --out \' ./\'
                 --format \' ALL\'
                 --prettyPrint''' odcInstallation: 'OWASP-DepCheck-12-1-2'
                
            }
        }
    }
}