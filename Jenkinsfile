pipeline {
  agent {
    node {
      label 'base'
    }

  }
  stages {
    stage('拉取代码') {
      agent none
      steps {
        container('base') {
          git(credentialsId: 'gitee-auth', url: 'https://gitee.com/cloudscientific/inventory.git', branch: 'master', changelog: true, poll: false)
        }

      }
    }

    stage('打包') {
      agent none
      steps {
        container('base') {
          sh 'mvn clean package -Dmaven.test.skip=true -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true'
        }
      }
    }

    stage('构建-推送') {
      agent none
      steps {
        container('base') {
          sh 'docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER .'
          sh 'docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest .'
          withCredentials([usernamePassword(credentialsId : 'aliyun-docker-hub' ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER'
            sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest'
          }

        }

      }
    }
    stage('部署') {
      agent none
      steps {
        container('base') {
          withCredentials([
            string(
              credentialsId: "$KUBECONFIG_CREDENTIAL_ID",
              variable: 'KUBECONFIG_CONFIG'
            )
          ]) {sh '''
          printf "%s" "$KUBECONFIG_CONFIG" > kubeconfig
          envsubst < ./deploy.yml | kubectl --kubeconfig=kubeconfig apply -f -'''}
        }
      }
    }
}
  environment {
    KUBECONFIG_CREDENTIAL_ID = 'demo-kubeconfig'
    REGISTRY = 'registry.cn-shanghai.aliyuncs.com'
    DOCKERHUB_NAMESPACE = 'wangchengwork'
    APP_NAME = 'inventory'
    APIFOX_ACCESS_TOKEN = 'apifox-auth'
  }
}