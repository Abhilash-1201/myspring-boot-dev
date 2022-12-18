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
       stage ('K8S Deploy') {
       steps { 
                kubernetesDeploy(
                    configs: 'springboot-lb.yaml',
                    kubeconfigId: 'k8s',
                    enableConfigSubstitution: true
                    )               
  
      }  
    }

    }
    post{
        always{
            emailext to: "abhilash.rl@cloudjournee.com",
            cc : "nayab.s@cloudjournee.com"
            subject: "INPUT: Build ${env.JOB_NAME}",
            body: "Awaiting for your input ${env.JOB_NAME} build no: ${env.BUILD_NUMBER} .Click below to promote to production\n${env.JENKINS_URL}job/${env.JOB_NAME}\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
             timeout(time: 60, unit: 'MINUTES'){
            input message: "Promote to Production?", ok: "Promote"
          }
        }
    }
 
}
