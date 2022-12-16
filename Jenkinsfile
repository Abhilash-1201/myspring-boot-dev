pipeline{
    agent any
    environment { registry = "519852036875.dkr.ecr.us-east-2.amazonaws.com/demo_project:latest"}
    tools {maven "MAVEN"}
    stages{
        stage('code checkout from GitHub'){
            steps{
                //check out code from the GitHub
                git branch: 'main', url: 'https://github.com/Abhilash-1201/myspring-boot-dev.git'
            }
        }
        //This stage gets all code Quality check from the GitHub Repository
        stage('Code Quality Check via SonarQube'){
            steps{
                script{
                    def scannerHome = tool 'sonarqube-scanner';
                    withSonarQubeEnv(credentialsId: 'sonarqube_access_token'){
                        if(fileExists("sonar-project.properties")) {
                         sh "${tool("sonarqube-scanner")}/bin/sonar-scanner"
                             
                         }  
                        
                    }
                }
            }
        }
        stage('Build') {
            steps {
                // Build the Maven code after analysis
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        // Build the docker image to store in to ECR
        stage('Building docker image')  {
         steps{
           script{
               dockerImage = docker.build registry
           }
         }
       }
        // Push the docker image in to ECR
       stage('Pushing to ECR') {
        steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 519852036875.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push 519852036875.dkr.ecr.us-east-2.amazonaws.com/demo_project:latest'
               }
           }
      
        }  
        stage ('Prompt check'){
           steps {
           mail to: 'rlabhilash1201@gmail.com',
                cc : 'rlabhilashabhi07@gamil.com'
               subject: "INPUT: Dev Test Build", 
               body: "testing"
                timeout(time: 60, unit: 'MINUTES'){
                    input message: "Promote to Production?", ok: "Promote"
                }
            }
        }
        
    }

    }
}
