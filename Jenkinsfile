pipeline {
    agent any
    
    parameters {
        // Choose an environment to deploy frond-end resources: 'dev', 'uat', or 'prod'.
        choice(choices: ['dev', 'uat', 'prod'], name: 'Environment', description: 'Please choose an environment.')

        // Plan is used for gengrating plan file. Apply is used to deploy or destroy resources.
        choice(choices: ['plan','apply'], name: 'plan_apply', description: 'Plan is used for gengrating plan file. Apply is used to deploy or destroy resources.')
        
        // Apply or destroy resources
        choice(choices: ['deploy', 'destroy'], name: 'Operation', description: 'Plan or apply or destroy resources.')
    }
    
    stages {
      stage('Check out code'){
        steps{
          git branch:'master', url:'https://github.com/liwenbo55/p3_Techscrum.tf.fe.git'                
        }
      }

      stage('IaC') {
        steps{
          script {
            withCredentials([
              [$class: 'AmazonWebServicesCredentialsBinding', 
               credentialsId: 'lawrence-jenkins-credential']
            ]){
              dir('app/techscrum_fe'){
                  
                  if (params.Environment in ['dev', 'uat', 'prod']) {
                      echo "Deploying front-end resources for ${params.Environment} environment."
                      
                      // Echo terraform vision
                      sh 'terraform --version'

                      // Terraform init
                      sh "terraform init -reconfigure -backend-config=backend_${params.Environment}.conf"

                      // Terraform plan
                      if (params.Operation == 'deploy') {
                          sh "terraform plan -var-file=${params.Environment}.tfvars -out=${params.Environment}_${params.Operation}_plan"
                      } else if (params.Operation == 'destroy') {
                          sh "terraform plan -var-file=${params.Environment}.tfvars -out=${params.Environment}_${params.Operation}_plan -destroy"
                      }
                                          
                      // Terraform actions
                      if (params.plan_apply == 'apply') {
                          // Terraform APPLY
                          sh "terraform apply '${params.Environment}_${params.Operation}_plan'"
                        } 
                      
                      sh 'ls -la'
                      
                      // 
                      sh "terraform show -no-color '${params.Environment}_${params.Operation}_plan' > '${params.Environment}_${params.Operation}_plan.txt' "                      
                          
                    } else {
                        error "Invalid environment: ${params.Environment}."
                    }                 
                  
                  // if (params.Environment in ['dev', 'uat', 'prod']) {
                  //     echo "Deploying front-end resources for ${params.Environment} environment."
                      
                  //     // Echo terraform vision
                  //     sh 'terraform --version'

                  //     // Terraform init
                  //     sh "terraform init -reconfigure -backend-config=backend_${params.Environment}.conf"

                  //     // Terraform actions
                  //     if (params.Operation == 'apply') {
                  //         // Terraform APPLY
                  //         sh "terraform plan -var-file=${params.Environment}.tfvars -out=${params.Environment}_${params.Operation}_plan"
                  //         sh "terraform apply '${params.Environment}_${params.Operation}_plan'"
                  //       } else if (params.Operation == 'destroy') {
                  //         // Terraform DESTROY 
                  //         sh "terraform plan -var-file=${params.Environment}.tfvars -out=${params.Environment}_${params.Operation}_plan -destroy"
                  //         sh "terraform apply '${params.Environment}_${params.Operation}_plan'"
                  //       } else if (params.Operation == 'plan') {
                  //         sh "terraform plan -var-file=${params.Environment}.tfvars -out=${params.Environment}_${params.Operation}_plan"
                  //       } 
                      
                  //     sh 'ls -la'
                          
                  //   } else {
                  //       error "Invalid environment: ${params.Environment}."
                  //   }
              }
            }
          }
        }
      }
    }
    post {
        success {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end terraform pipeline successed.",
                body: "Front-end resources for ${params.Environment} environment have been successfully ${params.Operation}ed. Please check the plan file.",
                attachLog: false,
                // attachmentsPattern: "**/'${params.Environment}_${params.Operation}_plan.txt'"
                attachmentsPattern: "**/dev_destroy_plan.txt"
            )
        }

        failure {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end terraform pipeline failed.",
                body: "Front-end resources for ${params.Environment} environment have failed to be ${params.Operation}ed. Please check logfile for more details.",
                attachLog: true
            )
        }
    }
}
