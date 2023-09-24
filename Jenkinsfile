pipeline {
   tools {
        maven 'Maven3'
    }
    agent any
    environment {
        registry = "923770093922.dkr.ecr.us-east-1.amazonaws.com/myrepo"
    }
   
    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/malibalakrishna/springbootapp.git']]])     
            }
        }
      stage ('Build') {
          steps {
            sh 'mvn clean install'           
            }
      }
      stage('Unit Test') {
            steps {
                echo '<--------------- Unit Testing started  --------------->'
                sh 'mvn surefire-report:report'
                echo '<------------- Unit Testing stopped  --------------->'
      }
    }
      stage('Sonar Analysis') {
         environment {
            scannerHome = tool 'sonar-scanner'
      }
      steps {
        echo '<--------------- Sonar Analysis started  --------------->'
        //         withSonarQubeEnv('sonar-cloud') {
        //         sh "${scannerHome}/bin/sonar-scanner"

        // }
        withSonarQubeEnv('sonar-cloud') {
          sh 'mvn clean package sonar:sonar -Dsonar.projectKey=shalinimeka_cicd-pipeline -Dsonar.organization=shalinimeka -Dsonar.sources=. -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=e36fc75e990d44af5d5cbb7db0202b309e661ba4'
          echo '<--------------- Sonar Analysis stopped  --------------->'
        }
      }
    }
         stage('Quality Gate') {
      steps {
        script {
          echo '<--------------- Quality Gate started  --------------->'
          timeout(time: 1, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
              error 'Pipeline failed due to the Quality gate issue'
            }
          }
          echo '<--------------- Quality Gate stopped  --------------->'
        }
      }
         }

      stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t myrepo .'
                }
            }
    }
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 923770093922.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker tag myrepo:latest 923770093922.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
                sh 'docker push 923770093922.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
         }
        }
      }
             stage('Deploy to EKS') {
            steps {
                script {
                    // Authenticate with the EKS cluster (ensure AWS credentials are configured)
                    sh 'aws eks --region us-east-1 update-kubeconfig --name demo-eks'
                    
                    // Apply Kubernetes manifest files to deploy your application
                     // sh "kubectl delete -f eks-deploy-k8s.yaml"
                      sh "kubectl apply -f eks-deploy-k8s.yaml"
                }
            }
        }
    } 
}
