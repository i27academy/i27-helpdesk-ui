pipeline {
    agent {
        label 'my-slave'
    }
    parameters {
        booleanParam(name: 'BUILD', defaultValue: true, description: "Run buid and push image")
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
            when {
                expression {
                    return params.BUILD
                }
            }
            steps {
                script {
                // Construct a full image with complete image name 
                env.IMAGE_NAME = env.REGISTRY_URL + "/" + env.IMAGE_REPOSITORY 

                echo "Using Registry: ${env.REGISTRY_URL}"
                echo "Using Repository: ${env.IMAGE_REPOSITORY}"
                echo "Using Image Tag: ${GIT_COMMIT}"
                echo "Full Image is: ${env.IMAGE_NAME}:${GIT_COMMIT}"
                //docker.io/devopswithcloudhub/i27-helpdesk-ui:COMMIT_ID
                }

            }
        }
        stage('Build Docker Image') {
            when {
                expression {
                    return params.BUILD
                }
            }
            steps {
                echo "Building the image"
                //sh "docker build -t  ${env.IMAGE_NAME}:${GIT_COMMIT} --build-arg NEXT_PUBLIC_API_BASE_URL=http://34.9.152.246:8080 ."
            }
        }
    }
}


