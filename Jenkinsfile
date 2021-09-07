pipeline {
  agent any
  environment {
    GIT_REPO_NAME = env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
    SOME_SECRET_KEY = credentials('some-secret-key')
  }
  stages {
    stage('Build image') {
      steps {
        sh 'echo $SOME_SECRET_KEY'
        sh 'docker build . -t "localhost:5000/${GIT_REPO_NAME}.${BRANCH_NAME}:${BUILD_NUMBER}"'
        sh 'docker push localhost:5000/${GIT_REPO_NAME}.${BRANCH_NAME}:${BUILD_NUMBER}'
        sh 'docker stop ${GIT_REPO_NAME}.${BRANCH_NAME} || true'
      }
    }

    stage('Run images') {
      steps {
        sh 'docker network create --driver bridge ${GIT_REPO_NAME}.${BRANCH_NAME}'
        sh 'docker run -d --rm --net ${GIT_REPO_NAME}.${BRANCH_NAME} --name postgres.${GIT_REPO_NAME}.${BRANCH_NAME} postgres:11 -v pgdata:/var/lib/postgresql/data'
        sh 'docker run -d --rm --net ${GIT_REPO_NAME}.${BRANCH_NAME} --name ${GIT_REPO_NAME}.${BRANCH_NAME} -p 3000 localhost:5000/${GIT_REPO_NAME}.${BRANCH_NAME}:${BUILD_NUMBER}'
        sh '''
          JENKINS_IMAGE_PORT=`docker port ${GIT_REPO_NAME}.${BRANCH_NAME} | egrep [0-9]+$ -o | head -1`
          echo "localhost:$JENKINS_IMAGE_PORT"
        '''
      }
    }


  }
}
