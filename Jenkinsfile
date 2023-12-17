pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "mloche" // replace this with your docker-id
MOVIES_EXAM_DB = "jenkins-exam-movies-db"
MOVIES_EXAM_APP = "jenkins-exam-movies-app"
CASTS_EXAM_APP = "jenkins-exam-casts-app"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages{
// build movie db

stage('cleaning former runs') {
steps{
sh """
docker rm -f movie-db-container
docker rm -f cast-db-container
docker rm -f exam-nginx
docker rm -f exam-movie-app
docker rm -f exam-casts-app
"""
}
}

stage('Deploy movie DB') {
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

// build nginx
stage('build nginx'){
	steps{
		sh """
		cd nginx
		docker build -t exam_nginx:$DOCKER_TAG	.
		"""
	}
}


stage('deploy nginx') {
	steps{
		sh '''
		docker run -d -p 8011:8080 --name exam-nginx -v ./nginx_config.conf:/etc/nginx/default.conf --ip 172.17.0.1 exam_nginx:$DOCKER_TAG
		'''
		}
	}

// build movie api

stage('build movie api') {
	steps{
		sh """
                cd movie-service
 		docker build -t $MOVIES_EXAM_APP:$DOCKER_TAG .
		"""
	}

}
// start movie api

stage('start movie API'){
	steps{
		sh '''docker run -d -p 8009:8000 --name exam-movie-app -v ./movie-service/:/app/ -e DATABASE_URI=postgresql://movie_db_username:movie_db_password@172.17.0.2/movie_db_dev \\
          -e CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/ --ip 172.17.0.4 $MOVIES_EXAM_APP:$DOCKER_TAG uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 '''
	}
}

// build casts api
stage('build cast api') {
        steps{
                sh """
                cd cast-service
                docker build -t $CASTS_EXAM_APP:$DOCKER_TAG .
                """
        }

}
// start movie api

stage('start casts API'){
        steps{
                sh ''' docker run -d -p 8010:8000 --name exam-casts-app -v ./casts-service/:/app/ -e DATABASE_URI=postgresql://movie_db_username:casts_db_password@172.17.0.3/casts_db_dev \\
                 --ip 172.17.0.5 $CASTS_EXAM_APP:$DOCKER_TAG uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 '''
                     }
}




// test api

// test movies
// test casts 

//Push images in docker hub
// push in git


}




post { // send email when the job has failed
    // ..
    failure {
        echo "This will run if the job failed"
        mail to: "loche.eu@gmail.com",
             subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
             body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
    }
    // ..
}
}
