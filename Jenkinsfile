pipeline {
  agent any

  environment {
    REMOTE_USER = 'user'
    REMOTE_HOST = '172.16.1.167'
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
        withCredentials([usernamePassword(credentialsId: 'win-ssh-creds', usernameVariable: 'SSH_USER', passwordVariable: 'SSH_PASS')]) {
          sh '''
          sshpass -p "$SSH_PASS" scp -o StrictHostKeyChecking=no -r deploy/* $SSH_USER@$REMOTE_HOST:"$REMOTE_PATH/"
          sshpass -p "$SSH_PASS" ssh -o StrictHostKeyChecking=no $SSH_USER@$REMOTE_HOST powershell -Command "
            cd '$env:REMOTE_PATH';
            npm install --omit=dev;
            npm run start;
          "
          '''
        }
      }
    }

    stage('Cleanup') {
      steps {
        sh 'rm -rf deploy .next node_modules'
      }
    }
  }
}
