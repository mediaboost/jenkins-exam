pipeline {
environment {
DOCKER_ID = "lantoki"
DOCKER_CAST_IMAGE = "jenkins-cast-service"
DOCKER_MOVIE_IMAGE = "jenkins-movie-service"
DOCKER_TAG = "v.${BUILD_ID}.0"
NODEPORT_DEV = "30001"
NODEPORT_QA = "30002"
NODEPORT_STAGING = "30003"
NODEPORT_PROD = "30004"
}
agent any
stages {
  stage('Docker Images Services Build'){
    steps {
      script {
      sh '''
        docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG ./cast-service
        docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG ./movie-service
      sleep 6
      '''
      }
    }
  }
  stage('Docker run'){
    steps {
      script {
      sh '''
      docker run -d -p 8001:8000 --name cast-service $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
      docker run -d -p 8002:8000 --name movie-service $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
      sleep 10
      '''
      }
    }
  }
  stage('Test Acceptance'){ 
    steps {
      script {
      sh '''
      curl localhost
      '''
      }
    }
  }
  stage('Docker Push'){
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }
    steps {
      script {
        sh '''
        docker login -u $DOCKER_ID -p $DOCKER_PASS
        docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
        docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
        '''
      }
    }
  }
  stage('Deploiement en dev'){
    environment {
    KUBECONFIG = credentials("config")
    }
    steps {
      script {
      sh '''
      rm -Rf ~/.kube/
      mkdir ~/.kube/
      ls
      cat $KUBECONFIG > ~/.kube/config

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install movie ./charts --values=values.yml --namespace dev --set service.nodePort=$NODEPORT_DEV

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install cast ./charts --values=values.yml --namespace dev --set service.nodePort=$NODEPORT_DEV
      
      '''
      }
    }
  }
  stage('Deploiement en qa'){
    environment {
    KUBECONFIG = credentials("config")
    }
    steps {
      script {
      sh '''
      rm -Rf ~/.kube/
      mkdir ~/.kube/
      ls
      cat $KUBECONFIG > ~/.kube/config

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install movie ./charts --values=values.yml --namespace qa --set service.nodePort=$NODEPORT_QA

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install cast ./charts --values=values.yml --namespace qa --set service.nodePort=$NODEPORT_QA
      
      '''
      }
    }
  }
  stage('Deploiement en staging'){
    environment {
    KUBECONFIG = credentials("config")
    }
    steps {
      script {
      sh '''
      rm -Rf ~/.kube/
      mkdir ~/.kube/
      ls
      cat $KUBECONFIG > ~/.kube/config

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install movie ./charts --values=values.yml --namespace staging --set service.nodePort=$NODEPORT_STAGING

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install cast ./charts --values=values.yml --namespace staging --set service.nodePort=$NODEPORT_STAGING
      
      '''
      }
    }
  }
  stage('Deploiement en prod'){
    when {
      branch 'master'
    }
    environment {
      KUBECONFIG = credentials("config")
      }
      steps {
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          rm -Rf ~/.kube/
          mkdir ~/.kube/
          ls
          cat $KUBECONFIG > ~/.kube/config

          cp charts/values.yaml values.yml
          sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE_IMAGE}+g" values.yml
          sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
          helm upgrade --install movie ./charts --values=values.yml --namespace prod --set service.nodePort=$NODEPORT_PROD

          cp charts/values.yaml values.yml
          sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
          sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
          helm upgrade --install cast ./charts --values=values.yml --namespace prod --set service.nodePort=$NODEPORT_PROD
          
          '''
        }
      }
    }
  }
}
