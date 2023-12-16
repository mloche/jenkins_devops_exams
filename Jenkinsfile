pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "mloche" // replace this with your docker-id
MOVIES_EXAM_DB = "jenkins-exam-movies-db"
MOVIES_EXAM_APP = "jenkins-exam-movies-app"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages{
# build movie db
# build movie api
# build nginx
# test api
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
