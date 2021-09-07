pipeline {
  agent any
  environment {
    GIT_REPO_NAME = env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
    SOME_SECRET_KEY = credentials('some-secret-key')
    IMAGE_NAME = 'nest-helloworld'
  }
  stages {
    stage('Run db') {
      steps {
        sh 'docker run -d --rm --name postgres.${GIT_REPO_NAME}.${BRANCH_NAME} postgres:11 -v pgdata:/var/lib/postgresql/data'
      }
    }

    stage('Build image') {
      steps {
        sh 'echo $SOME_SECRET_KEY'
        sh 'docker build . -t "localhost:5000/${IMAGE_NAME}:${GIT_REPO_NAME}.${BRANCH_NAME}.${BUILD_NUMBER}"'
        sh 'docker push localhost:5000/${IMAGE_NAME}:${GIT_REPO_NAME}.${BRANCH_NAME}.${BUILD_NUMBER}'
        sh 'docker stop ${IMAGE_NAME} || true'
      }
    }
    
    stage('Run images') {
      steps {
        sh 'docker run -d --rm --name postgres.${GIT_REPO_NAME}.${BRANCH_NAME} postgres:11 -v pgdata:/var/lib/postgresql/data'
        sh 'docker run -d --rm --name ${IMAGE_NAME} -p 3000 localhost:5000/${IMAGE_NAME}:${GIT_REPO_NAME}.${BRANCH_NAME}.${BUILD_NUMBER}'
        sh '''
          JENKINS_IMAGE_PORT=`docker port ${IMAGE_NAME} | egrep [0-9]+$ -o | head -1`
          echo "localhost:$JENKINS_IMAGE_PORT"
        '''
      }
    }


  }
}
