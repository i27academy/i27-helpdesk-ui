pipeline {
    agent {
        label 'my-slave'
    }
    environment {
        // Currently i am using docker hub registry
        REGISTRY_URL = "docker.io"
        // later u changed to jfrog ====> jfrog.hsbc.com
        IMAGE_REPOSITORY = "devopswithcloudhub/i27-helpdesk-ui"

        // docker.io/devopswithcloudhub/i27-helpdesk-ui:tagname
        // docker.io/devopswithcloudhub/i27-helpdesk-ui:84285da
 
    }
    stages {
        stage ('Prepare Tag') {
            steps {
                // Construct a full image with complete image name 
                env.IMAGE_NAME = env.REGISTRY_URL + "/" + env.IMAGE_REPOSITORY 

                echo "Using Registry: ${env.REGISTRY_URL}"
                echo "Using Repository: ${env.IMAGE_REPOSITORY}"
                echo "Using Image Tag: ${GIT_COMMIT}"
                echo "Full Image is: ${env.IMAGE_NAME}:${GIT_COMMIT}"
                //docker.io/devopswithcloudhub/i27-helpdesk-ui:COMMIT_ID
            }
        }
        // stage('Build Docker Image') {
        //     steps {
        //         sh "docker build -t  ${env.IMAGE_NAME}:${GIT_COMMIT} --build-arg NEXT_PUBLIC_API_BASE_URL=http://34.121.17.29:8080 ."
        //     }
        // }
    }
}


