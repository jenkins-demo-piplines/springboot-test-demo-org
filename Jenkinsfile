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

        stage('Build Docker Image'){
            steps{
                sh 'docker build -t slpavaniv/springboot-test-demo-org:$GIT_COMMIT .'
            }
        }

                stage('Push Docker Image'){
            steps{
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: "") {
                    sh 'docker push slpavaniv/springboot-test-demo-org:$GIT_COMMIT'
                    }
            }
               
            }

            stage('Deploy -AWS EC2'){
                when {
                    branch 'feature/*'
                   }

                steps{
                    script{
                  sshagent(['aws-dev-deploy-ec2-instance']) {
                    sh '''
                     ssh -o StrictHostKeyChecking=no ubuntu@13.220.194.23 "
                    
                     if docker ps -a | grep -q "springboot-test-demo-org"; then
                     echo "Container found. Stopping.."
                     docker stop "springboot-test-demo-org" && docker rm "springboot-test-demo-org"
                     echo "Container stopped and removed."
                     fi
                     

                     docker run --name springboot-test-demo-org -p 8080:8080 slpavaniv/springboot-test-demo-org:$GIT_COMMIT
                     "
                    '''
                     }
                }
                }
            }
        

//           stage('Sonarqube Analysis') {
//             steps {
//                 sh '''mvn clean verify sonar:sonar \
//   -Dsonar.projectKey=springboot-test-demo-org \
//   -Dsonar.host.url=http://localhost:9001 \
//   -Dsonar.login=sqp_2bf68aa9cce2d8c4a44b939f087a6150a46d26e5 '''
//             }
//             // waitForQualityGate abortPipeline: true
//         }


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