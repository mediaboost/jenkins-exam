pipeline {
environment {
DOCKER_ID = "lantoki"
DOCKER_CAST_IMAGE = "jenkins-cast-service"
DOCKER_MOVIE_IMAGE = "jenkins-movie-service"
DOCKER_TAG = "v.${BUILD_ID}.0"
NODEPORT_MOVIE_DEV = "30011"
NODEPORT_MOVIE_QA = "30012"
NODEPORT_MOVIE_STAGING = "30013"
NODEPORT_MOVIE_PROD = "30014"
NODEPORT_CAST_DEV = "30015"
NODEPORT_CAST_QA = "30016"
NODEPORT_CAST_STAGING = "30017"
NODEPORT_CAST_PROD = "30018"
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
      docker rm -f cast-service movie-service || true
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
      curl -f http://localhost:8001/docs || true
      curl -f http://localhost:8002/docs || true
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
      helm upgrade --install movie ./charts --values=values.yml --namespace dev --set service.nodePort=$NODEPORT_MOVIE_DEV

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install cast ./charts --values=values.yml --namespace dev --set service.nodePort=$NODEPORT_CAST_DEV
      
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
      helm upgrade --install movie ./charts --values=values.yml --namespace qa --set service.nodePort=$NODEPORT_MOVIE_QA

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install cast ./charts --values=values.yml --namespace qa --set service.nodePort=$NODEPORT_CAST_QA
      
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
      helm upgrade --install movie ./charts --values=values.yml --namespace staging --set service.nodePort=$NODEPORT_MOVIE_STAGING

      cp charts/values.yaml values.yml
      sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
      sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
      helm upgrade --install cast ./charts --values=values.yml --namespace staging --set service.nodePort=$NODEPORT_CAST_STAGING
      
      '''
      }
    }
  }
  stage('Deploiement en prod'){
    when {
      expression {
        return env.BRANCH_NAME == 'master' ||
              env.GIT_BRANCH == 'master' ||
              env.GIT_BRANCH == 'origin/master'
      }
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
          helm upgrade --install movie ./charts --values=values.yml --namespace prod --set service.nodePort=$NODEPORT_MOVIE_PROD

          cp charts/values.yaml values.yml
          sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST_IMAGE}+g" values.yml
          sed -i "s+tag:.*+tag: \"${DOCKER_TAG}\"+g" values.yml
          helm upgrade --install cast ./charts --values=values.yml --namespace prod --set service.nodePort=$NODEPORT_CAST_PROD
          
          '''
        }
      }
    }
  }
}
