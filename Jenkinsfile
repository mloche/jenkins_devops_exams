pipeline {
  environment { 
    DOCKER_ID = "mloche" // replace this with your docker-id
    MOVIES_EXAM_DB = "jenkins-exam-movies-db"
    MOVIES_EXAM_APP = "jenkins-exam-movies-app"
    CASTS_EXAM_APP = "jenkins-exam-casts-app"
    NGINX_EXAM_APP = "jenkins-exam-nginx-app"
    DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
  }
  agent any // Jenkins will be able to select all available agents
  stages{
    stage('Cleaning former runs') {
      steps{
        sh """
        docker rm -f movie-db-container
        docker rm -f cast-db-container
        docker rm -f exam-nginx
        docker rm -f exam-movie-app
        docker rm -f exam-casts-app
        docker rm -f exam-test-nginx
        docker rm -f exam-test-movies
        docker rm -f exam-test-casts
        """
      }
    }

  stage('echo branch name'){
    when{
      expression{
        return env.GIT_BRANCH == "origin/master"
      }
    }
    steps{
      echo "master"
    }
  }


  stage('Deploy the movie DB') {
    steps {
      sh '''docker run -d --name movie-db-container -v postgres_data_movie:/var/lib/postgresql/data/ -e POSTGRES_USER=movie_db_username \\
              -e POSTGRES_PASSWORD=movie_db_password -e POSTGRES_DB=movie_db_dev --ip 172.17.0.2 postgres:12.1-alpine'''
    }
  }

  stage('Deploy cast DB') {
    steps {
      sh'''docker run -d --name cast-db-container -v postgres_data_casts:/var/lib/postgresql/data/ -e POSTGRES_USER=cast_db_username \\
          -e POSTGRES_PASSWORD=cast_db_password -e POSTGRES_DB=cast_db_dev --ip 172.17.0.3 postgres:12.1-alpine'''
    }
  }

  stage('build nginx'){
    steps{
      sh """
      cd nginx
      docker build -t $DOCKER_ID/$NGINX_EXAM_APP:$DOCKER_TAG .
      """
    }
  }


  stage('deploy nginx') {
    steps{
      sh '''
      docker run -d -p 8011:8080 --name exam-nginx -v ./nginx_config.conf:/etc/nginx/default.conf --ip 172.17.0.4 $DOCKER_ID/$NGINX_EXAM_APP:$DOCKER_TAG
      '''
    }
  }

// build movie api

  stage('build movie api') {
    steps{
      sh """
      cd movie-service
      docker build -t $DOCKER_ID/$MOVIES_EXAM_APP:$DOCKER_TAG .
      """
    }
  }

// start movie api
  stage('start movie API'){
    steps{
      sh '''docker run -d -p 8009:8000 --name exam-movie-app -v ./movie-service/:/app/ -e DATABASE_URI=postgresql://movie_db_username:movie_db_password@172.17.0.2/movie_db_dev \\
          -e CAST_SERVICE_HOST_URL=http://172.17.0.6:8000/api/v1/casts/ --ip 172.17.0.5 $DOCKER_ID/$MOVIES_EXAM_APP:$DOCKER_TAG uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 '''
    }
  }

// build casts api
  stage('build cast api') {
    steps{
      sh """
      cd cast-service
      docker build -t $DOCKER_ID/$CASTS_EXAM_APP:$DOCKER_TAG .
      """
    }
  }

// start movie api
  stage('start casts API'){
    steps{
      sh ''' docker run -d -p 8010:8000 --name exam-casts-app -v ./cast-service/:/app/ -e DATABASE_URI=postgresql://cast_db_username:cast_db_password@172.17.0.3/cast_db_dev \\
                 --ip 172.17.0.6 $DOCKER_ID/$CASTS_EXAM_APP:$DOCKER_TAG uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 '''
    }
  }


// test nginx
  stage('Testing Nginx'){
    steps{sh ''' docker run -d --name exam-test-nginx curlimages/curl -L -v http://172.17.0.4'''}
  }

// test movies
  stage('Testing movies api'){
    steps{sh ''' docker run -d --name exam-test-movies curlimages/curl -L -v http://172.17.0.5:8000/api/v1/movies/status'''}
  }

// test casts 

  stage('Testing casts api'){
    steps{sh ''' docker run -d --name exam-test-casts curlimages/curl -L -v http://172.17.0.6:8000/api/v1/casts/status'''}
  }


//Push images in docker hub
  stage('Push images in docker Hub'){
    environment{
      DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }
    steps{
      sh '''
      docker login -u $DOCKER_ID -p $DOCKER_PASS
      docker push $DOCKER_ID/$NGINX_EXAM_APP:$DOCKER_TAG
      docker push $DOCKER_ID/$CASTS_EXAM_APP:$DOCKER_TAG
      docker push $DOCKER_ID/$MOVIES_EXAM_APP:$DOCKER_TAG
      '''
    }
  }


/*  stage('Stop old Dev'){
    environment{
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      timeout: time(minutes: 5){
                input message: 'Want to stop old dev?', ok: 'Yes',
//                  parameters: [booleanParam(name: 'stop_dev', defaultValue: true)]
      }
      script {
  //      if(params.stop_dev) {
          sh '''
          helm uninstall jenkins --namespace dev
          '''
    //    }
      }
    }
  }
*/

  stage('Deploy Dev'){
    environment{
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
        sh '''
        helm upgrade --install jenkins jenkins-exam/ --values=./jenkins-exam/values.yaml --namespace dev
        '''
      }
    }
  }

  stage('Deploiement en prod'){
    environment{
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    when{
      expression{
        return env.GIT_BRANCH == "origin/master"
      }
    }
    steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
      timeout(time: 15, unit: "MINUTES"){        
      input message: 'Want to deploy in prod ', ok: 'Yes',
      //            parameters: [booleanParam(name: 'deploy_prod', defaultValue: false)]
      }                   
    
      script{
  //      if(params.deploy_prod){
          sh '''
          helm upgrade --install jenkins jenkins-exam/ --values=./jenkins-exam/values.yaml --namespace prod
          '''
       // }
      }
    }
  }
//closing Stages section
}




  post { // send email when the job has failed
    failure {
        echo "This will run if the job failed"
        mail to: "loche.eu@gmail.com",
             subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
             body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
    }
  }
// Closing Pipeline
}
