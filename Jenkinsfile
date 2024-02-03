pipeline {
  agent any
  tools {
  
  maven 'MAVEN'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/DevOps-Module/sept23.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }
         
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
                
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }
				
              }
            }
      }


    stage('Copy Dockerfile & Playbook to Staging Server') {
            
            steps {
                  sshagent(['ssh_agent']) {
                       sh "chmod 400 mo-ohio-kp.pem"
                       sh "ls -lah"
                      //sh "scp -i mo-ohio-kp.pem -o StrictHostKeyChecking=no Dockerfile ubuntu@3.142.129.62:/home/ubuntu"
                        sh "scp -i mo-ohio-kp.pem -o StrictHostKeyChecking=no dockerfile ubuntu@3.142.129.62:/home/ubuntu"
                        sh "scp -i mo-ohio-kp.pem -o StrictHostKeyChecking=no deploy-2-dockerhub.yaml ubuntu@3.142.129.62:/home/ubuntu"
                    }
                }
            
        } 

    stage('Build Container Image') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i mo-ohio-kp.pem -o StrictHostKeyChecking=no ubuntu@34.194.41.127 -C \"ansible-playbook -vvv -e build_number=${BUILD_NUMBER} deploy-2-dockerhub.yaml\""
                        
                    }
                }
        } 

    stage('Copy Deployment & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "scp -i mo-ohio-kp.pem -o StrictHostKeyChecking=no k8s-deployment.yaml ubuntu@3.136.6.38:/home/ubuntu"
                      //sh "scp -i mo-ohio-kp.pem -o StrictHostKeyChecking=no k8s-deployment.yaml ubuntu@3.136.6.38:/home/ubuntu"
                    }
                }
            
        } 

    stage('Waiting for Approvals') {
            
        steps{

				input('Test Completed ? Please provide  Approvals for Prod Release ?')
			  }
            
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['ssh_key']) {
                        //sh "ssh -i mo-ohio-kp.pem -o StrictHostKeyChecking=no ubuntu@3.136.6.38 -C \"kubectl set image deployment/ranty customcontainer=mobanntechnologies/july-set:${BUILD_NUMBER}\""
                        //sh "ssh -i mo-ohio-kp.pem -o StrictHostKeyChecking=no ubuntu@3.136.6.38 -C \"kubectl delete deployment ranty && kubectl delete service ranty\""
                        sh "ssh -i mo-ohio-kp.pem -o StrictHostKeyChecking=no ubuntu@3.136.6.38 -C \"kubectl apply -f k8s-deployment.yaml\""
                        //sh "ssh -i mo-ohio-kp.pem -o StrictHostKeyChecking=no ubuntu@3.136.6.386 -C \"kubectl apply -f service.yaml\""
                    }
                }
            
        } 
         
   } 
}



