def project = 'esparta86project'
def  appName = 'php-app'
def  appNameI = 'php-app'
def  serviceName = 'app'
def  versImage = 'v2' 
/* Define which version we use in order to build a new images in the future,it could be stable
def  feSvcNameCanary = "${serviceName}-app-service-lb-testing"*/
def  feSvcNameProduction = "${serviceName}-service-lb-production"
//def  feSvcNameDev = "${appName}-frontend-dev"

// ----- In each execution of pipeline this generates a new images ----------------
def  imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

// ----- Define the url of image in order to pull ----------------------------------
def  imageContainerBase = "gcr.io/${project}/${appName}:${versImage}"

// ----- set the name of container. It should be the same as deployment.yaml -------
// ----- In each execution of pipeline this generates a new images ----------------

def  nameImage = "web"

// ----- set service account Jenkins -----------------------------------------------
def  serviceAccount = 'cd-jenkins'

pipeline {
  agent {
    kubernetes {
      label 'app-production-git'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name:  "${nameImage}"
    image: "${imageContainerBase}"
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('Test') {
      steps {
        container('web') {
          sh """
            cd /var/www/html
            echo index.php
          """
        }
      }
    }
    stage('Build and push image with Container Builder') {
      steps {
        container('gcloud') {
          sh "PYTHONUNBUFFERED=1 gcloud container builds submit -t ${imageTag} ."
        }
      }
    }
    stage('Deploy Testing') {
      // Canary branch
      when { branch 'testing' }
      steps {
        container('kubectl') {
          // Change deployed image in canary to the one we just built
//         sh("sed -i.bak 's#gcr.io/ti-is-devenv-01/app-grol:v7#${imageTag}#' ./k8s/testing/*.yaml")
//         sh("kubectl --namespace=production-grol apply -f k8s/services/testing.yaml")
//          sh("kubectl --namespace=production-grol apply -f k8s/testing/")
//          sh("echo http://`kubectl --namespace=production-grol get service/${feSvcNameCanary} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcNameCanary}")
        }
      }
    }
    stage('Deploy Production') {
      // Production branch
      when { branch 'master' }
      steps{
        container('kubectl') {
        // Change deployed image in canary to the one we just built
          sh("sed -i.bak 's#gcr.io/esparta86project/php-app:v2#${imageTag}#' ./k8s/production/*.yaml")
          sh("kubectl --namespace=production apply -f k8s/services/servicesLB.yaml")
          sh("kubectl --namespace=production apply -f k8s/production/deployment.yaml")
          sh("echo http://`kubectl --namespace=production get service/${feSvcNameProduction} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcNameProduction}")
        }
      }
    }
    stage('Deploy Dev') {
      // Developer Branches
      when {
        not { branch 'master' }
        not { branch 'testing' }
      }
      steps {
        container('kubectl') {
          // Create namespace if it doesn't exist
//          sh("kubectl get ns ${env.BRANCH_NAME} || kubectl create ns ${env.BRANCH_NAME}")
         // Don't use public load balancing for development branches
//          sh("sed -i.bak 's#LoadBalancer#ClusterIP#' ./k8s/services/frontend.yaml")
//          sh("sed -i.bak 's#gcr.io/ti-is-devenv-01/app-grol:v1#${imageTag}#' ./k8s/dev/*.yaml")
//          sh("kubectl --namespace=${env.BRANCH_NAME} apply -f k8s/services/")
//          sh("kubectl --namespace=${env.BRANCH_NAME} apply -f k8s/dev/")
//          echo 'To access your environment run `kubectl proxy`'
//          echo "Then access your service via http://localhost:8001/api/v1/proxy/namespaces/${env.BRANCH_NAME}/services/${feSvcNameDev}:80/"
        }
      }
    }
  }
}

