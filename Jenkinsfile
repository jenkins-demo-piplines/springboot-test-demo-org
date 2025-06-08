pipeline{
    agent any

    tools{
        maven 'MVN387'
    }

    environment{
        MYSQL_URL = 'test'
    }

    options {
  disableConcurrentBuilds abortPrevious: true
  disableResume()
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
                            options {
                      timestamps()
                         }
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
                  junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'

                  publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', 
                  reportFiles: 'dependency-check-jenkins.html', 
                  reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                  
                
            }
        }

           }
        }
        stage('Unit Test'){
                    options {
                    retry(2)
                    }

            steps{
                withCredentials([usernamePassword(credentialsId: 'mysql-db-credentials', passwordVariable: 'MYSQL_PASSWORD', usernameVariable: 'MYSQL_USERNAME')]) {
       sh 'mvn test'
}
          junit allowEmptyResults: true, keepProperties: true, testResults: 'target/surefire-reports/*.xml'
              
            }
        }

     
    }
}