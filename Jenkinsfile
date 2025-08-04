pipeline {
  agent any

  environment {
    REMOTE_USER = 'lpdgi-syahrulsetiawan'
    REMOTE_HOST = '172.16.1.167'
    // REMOTE_PATH = 'C:/apps/next-app'
    REMOTE_PATH = 'C:/Users/user/Documents/Repositories/learn/next-js-demo'
  }

  triggers {
    githubPush()
  }

  stages {
    stage('Check Branch') {
      when {
        not {
          branch 'development'
        }
      }
      steps {
        echo 'Not on development branch — skipping pipeline.'
        script {
          currentBuild.result = 'SUCCESS'
          return
        }
      }
    }

    stage('Build') {
      steps {
        sh 'yarn install && yarn build'
      }
    }

    stage('Prepare Deploy Folder') {
      steps {
        sh '''
        rm -rf deploy
        mkdir -p deploy
        cp -r .next public package.json deploy/
        '''
      }
    }

    stage('Deploy via SSH') {
      steps {
        sh '''
        scp -r deploy/* $REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH/
        ssh $REMOTE_USER@$REMOTE_HOST powershell -Command "
          cd $env:REMOTE_PATH;
          npm install --omit=dev;
          npm run start;
        "
        '''
      }
    }

    stage('Cleanup') {
      steps {
        sh 'rm -rf deploy'
      }
    }
  }
}
