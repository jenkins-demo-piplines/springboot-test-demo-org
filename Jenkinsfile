pipeline{
    agent any

    tools{
        maven 'MVN387'
    }

    environment{
        MYSQL_URL = 'test'
        MYSQL_DB_CREDS = credentials('mysql-db-credentials')
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
            sh 'echo Colon-Separated $MYSQL_DB_CREDS'
            sh 'echo Username - $MYSQL_DB_CREDS_USR'
            sh 'echo Password - $MYSQL_DB_CREDS_PSW'
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
            }
        }

           }
        }
        stage('Unit Test'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'mysql-db-credentials', passwordVariable: 'MYSQL_PASSWORD', usernameVariable: 'MYSQL_USERNAME')]) {
       sh 'mvn test'
}
          
              
            }
        }

          stage('Sonarqube Analysis') {
            steps {
                sh '''mvn clean verify sonar:sonar \
  -Dsonar.projectKey=springboot-test-demo-org \
  -Dsonar.host.url=http://localhost:9001 \
  -Dsonar.login=sqp_2bf68aa9cce2d8c4a44b939f087a6150a46d26e5 '''
            }
        }


        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('ServerNameSonar') {
        //             bat '''mvn clean verify sonar:sonar -Dsonar.projectKey=ProjectNameSonar -Dsonar.projectName='ProjectNameSonar' -Dsonar.host.url=http://localhost:9000''' //port 9000 is default for sonar
        //             echo 'SonarQube Analysis Completed'
        //         }
        //     }
        // }

     
    }

    post{
        always{
            junit allowEmptyResults: true, keepProperties: true, testResults: 'target/surefire-reports/*.xml'
            junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', 
            reportFiles: 'dependency-check-jenkins.html', 
            reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}