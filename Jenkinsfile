def targetK8SEnv = "dev"
def targetK8SCluster= "dev"

pipeline {


  agent any

  stages {
  
    
    stage("Deploy") {
        input {
            message "Ready to deploy?"
            ok "Yes"
            parameters {
                string(name: "DEPLOY_ENV", defaultValue: "prod")
            }
        }
        steps {
            echo "Deploy to the ${DEPLOY_ENV} environment."
        }
    }
    
    
    stage("Git Clone"){
        
      steps {
        git credentialsId: 'GIT_HUB_CREDENTIALS', url: 'https://github.com/nirajvishwakarma/jhook-springboot.git'
      }
    }
    
    
    stage('Gradle Build') {
      steps {
       sh './gradlew build'
      }
 
    }
    
    
    stage("Docker build"){
      steps {
        sh 'docker version'
        sh 'docker build -t jhooq-docker-demo .'
        sh 'docker image list'
        sh 'docker tag jhooq-docker-demo nirajvishwakarma/jhooq-docker-demo:jhooq-docker-demo'
      }
    }
    
    
    stage("Docker Login"){
      steps {
        withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'PASSWORD')]) {
            sh 'docker login -u nirajvishwakarma -p $PASSWORD'
        }
      }
    }
    
    
    stage("Push Image to Docker Hub"){
      steps {
        sh 'docker push  nirajvishwakarma/jhooq-docker-demo:jhooq-docker-demo'
      }
    }




  
  

    stage('Interactive') {

      agent any

      steps {
        script {
          timeout ( time: 2, unit: "HOURS" ) {
            userAns = input(
              message: "Where to deploy to?",
              parameters: [choice(choices: ['dev', 'sit', 'demo', 'prod'],
              description: 'k8s env',
              name: 'deployTo')]
            )
          }

          switch(userAns) {
            case ["dev", "sit"]:
              targetK8SCluster = "dev"
              break
            case ["demo", "prod"]:
              targetK8SCluster = "prod"
              break
            default:
              error("Unknown error")
          }

          targetK8SEnv = userAns

          currentBuild.displayName = "${env.BUILD_ID}-${targetK8SEnv}-${env.TAG_NAME}"
        }
      }
    }
    
    
    stage ('K8S Deploy') {
        parallel {
                stage('deploy to development') {
                    when {
                      expression { targetK8SCluster == "dev" }
                    }
                    steps {
                        sh 'aws eks --region us-east-1 update-kubeconfig --name niraj-eks1'
                        kubernetesDeploy(configs: "k8s-spring-boot-deployment.yml", kubeconfigId: "dev")
                    }
                }
                stage('deploy to production') {
                    when {
                        expression { targetK8SCluster == "prod" }
                    }
                    steps {
                        sh 'aws eks --region us-east-1 update-kubeconfig --name niraj-eks3'
           	            kubernetesDeploy(configs: "k8s-spring-boot-deployment.yml", kubeconfigId: "prod")
                    }
                }
            }
    }
    
    
    
    stage("Completed") {
        steps {
            echo "Deployment Completed to the ${targetK8SCluster} environment."
        }
    }

    
    
    
  }

}
