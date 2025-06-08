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

        stage('Dependency Scanning'){

           parallel{

           stage('Dependency Audit'){
            steps{
            sh 'echo Dependency audit'
            }
           }

            stage('OWASP Dependency Check'){

            steps{
                // dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'OWASP-DepCheck-12-1-2'
               
                dependencyCheck additionalArguments: '''
                --scan \'./\'
                 --out \'./\'
                 --format \'ALL\'
                 --prettyPrint''' ,odcInstallation: 'OWASP-DepCheck-12-1-2'
                  dependencyCheckPublisher failedTotalCritical:1, pattern: 'dependency-check-report.xml',stopBuild: true
                  publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: '.', 
                  reportFiles: 'dependency-check-jenkins.html', 
                  reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                
            }
        }

           }
        }

     
    }
}