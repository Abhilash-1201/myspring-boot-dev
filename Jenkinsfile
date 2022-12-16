pipeline{
    agent any
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
               //dockerImage = docker.build registry + ":$BUILD_NUMBER"
               sh 'docker build -t springbootapp:latest .'
           }
         }
       }
       
    }
}
