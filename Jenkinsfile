pipeline {
    agent {
        label 'my-slave'
    }
    parameters {
        booleanParam(name: 'BUILD', defaultValue: true, description: "Run buid and push image")
        choice(name: 'TARGET_ENV', choices: ['dev', 'test', 'stage', 'prod'], description: 'Target environment for API url')
    }
    environment {
        // Currently i am using docker hub registry
        REGISTRY_URL = "docker.io"
        // later u changed to jfrog ====> jfrog.hsbc.com
        IMAGE_REPOSITORY = "devopswithcloudhub/i27-helpdesk-ui"
        // calling my docker creds into a variable
        REGISTRY_CREDENTIALS_ID = credentials('docker-credentials')

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

                // Setting the gateway url on the TARGET_ENV selection
                switch(params.TARGET_ENV)  {
                    case 'dev': 
                        env.NEXT_PUBLIC_API_BASE_URL = 'http://34.9.152.246:8080'
                        break
                    case 'test': 
                        env.NEXT_PUBLIC_API_BASE_URL = 'http://test-gateway.i27helpdesk.in'
                        break
                    case 'stage':
                        env.NEXT_PUBLIC_API_BASE_URL = 'http://stage-gateway.i27helpdesk.in'
                        break
                    case 'prod':
                        env.NEXT_PUBLIC_API_BASE_URL = 'http://gateway.i27helpdesk.in'
                        break
                }

                echo "Using Registry: ${env.REGISTRY_URL}"
                echo "Using Repository: ${env.IMAGE_REPOSITORY}"
                echo "Using Image Tag: ${GIT_COMMIT}"
                echo "Full Image is: ${env.IMAGE_NAME}:${GIT_COMMIT}"
                //docker.io/devopswithcloudhub/i27-helpdesk-ui:COMMIT_ID
                }
            }
        }
        stage ('SonarQube Analysis'){
            when {
                expression {
                    return params.BUILD
                }
            }
            steps {
                script {
                   def scannerHome = tool 'SonarQubeScanner'
                   withSonarQubeEnv('SonarQube'){
                    sh "${scannerHome}/bin/sonar-scanner"
                   }
                }


                // sonar-scanner \
                //     -Dsonar.host.url=http://SONARQUBE_PUBLIC_IP:9000 \
                //     -Dsonar.login=<PASTE_YOUR_TOKEN_HERE>
                    // we need to have sonarqube server 
                    // install SonarQube Scanner plugin in jenkins
                    // we need to have sonar-scanner installed on slave or we can use tools section in jenkins to install sonar-scanner and call it here
                    // we need to have sonar-scanner plugin installed on jenkins 
            }
        }
        stage ('Quality Gate') {
            when {
                expression {
                    return params.BUILD
                }
            }
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    script {
                        waitForQualityGate abortPipeline: true
                    }
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
                sh "docker build -t  ${env.IMAGE_NAME}:${GIT_COMMIT} --build-arg NEXT_PUBLIC_API_BASE_URL=${env.NEXT_PUBLIC_API_BASE_URL} ."
            }
        }
        stage ('Push Image') {
            when {
                expression {
                    return params.BUILD
                }
            }
            steps {
                echo "********************************* Docker Login *****************************"
                sh "docker login -u ${env.REGISTRY_CREDENTIALS_ID_USR} -p ${env.REGISTRY_CREDENTIALS_ID_PSW} ${env.REGISTRY_URL}"
                echo "********************************* Docker Push *****************************"
                sh "docker push ${env.IMAGE_NAME}:${GIT_COMMIT}"
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline Completed Succesfully'
        }
        failure {
            echo 'Pipeline Failed, check jenkins logs'
        }
    }
}


