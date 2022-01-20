pipeline {
      agent any
      environment {
           TWISTLOCK_TOKEN = credentials("TWISTLOCK_TOKEN")
        }
        
  stages {
    stage('Clone Github repository') {
      steps {
        checkout scm
      }
    }
    stage('Build Docker Image') {    
      steps {
        script {      
          app = docker.build("akhng999/vulnerablewebapp") 
        }
      } 
      post {
        success {
          archiveArtifacts 'target/*.hpi,target/*.jpi'
        }
      }
    }
    stage('Scan container before pushing to Dockerhub') {    
      steps {
        script {      
          try {
            /* checkpoint cloudguard
            sh 'docker save akhng999/vulnerablewebapp -o vwa.tar' 
            sh './shiftleft image-scan -i ./vwa.tar -t 1800'
            */
            sh '''
              chmod +x twistcli
              ./twistcli images scan \
                --address https://us-east1.cloud.twistlock.com/us-2-158255088 \
                --token ${TWISTLOCK_TOKEN} \
                --details \
                akhng999/vulnerablewebapp          
            '''
          } catch (Exception e) {
            echo "Security Test Failed" 
            env.flagError = "true"  
          }
        }
      }
      post {
        always {
          junit '**/twistlock-reports/**/*.xml'
        }
      }
    }
    stage('Dockerhub Approval Request') {
      when {
        expression { env.flagError == "true" }
      }
      steps {
        script {
          def userInput = input(id: 'confirm', message: 'This containers contains vulnerabilities. Push to Dockerhub?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Approve Code to Proceed', name: 'approve'] ])
        }
      }
    }   
    stage('Deploy App to Dockerhub') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            app.push("latest")
          }
        }
      }              
    }
  }
}