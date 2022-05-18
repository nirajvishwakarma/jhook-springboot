pipeline {
  agent any
  
  stages {  
    stage("Git Clone"){
 
        git credentialsId: 'GIT_HUB_CREDENTIALS', url: 'https://github.com/nirajvishwakarma/jhook-springboot.git'
    }
    
    stage('Gradle Build') {
 
       sh './gradlew build'
 
    }
    
    stage("Docker build"){
        sh 'docker version'
        sh 'docker build -t jhooq-docker-demo .'
        sh 'docker image list'
        sh 'docker tag jhooq-docker-demo nirajvishwakarma/jhooq-docker-demo:jhooq-docker-demo'
    }
 
    
    stage("Docker Login"){
        withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'PASSWORD')]) {
            sh 'docker login -u nirajvishwakarma -p $PASSWORD'
        }
    }
    
    stage("Push Image to Docker Hub"){
        sh 'docker push  nirajvishwakarma/jhooq-docker-demo:jhooq-docker-demo'
    }
    
    
    stage('Deliver for development') {
        when {
            branch 'development' 
        }
        steps {
            input message: 'Finished using the web site? (Click "Proceed" to continue)'
        }
    }
    stage('Deploy for production') {
        when {
            branch 'production'  
        }
        steps {
            input message: 'Finished using the web site? (Click "Proceed" to continue)'
        }
    }
    
    
 #    stage ('K8S Deploy') {
 #      
 #               kubernetesDeploy(
 #                   configs: 'k8s-spring-boot-deployment.yml',
 #                   kubeconfigId: 'k8s',
 #                   enableConfigSubstitution: true
 #                   )               
 #       }
  }
}
