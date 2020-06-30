pipeline {
  agent {
    docker {
      image 'jekyll/jekyll'
      args '''-v $PWD:/srv/jekyll
-it'''
    }

  }
  stages {
    stage('Build') {
      steps {
        sh 'jekyll build'
      }
    }

  }
}