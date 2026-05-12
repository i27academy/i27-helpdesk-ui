pipeline {
    agent {
        // label 'my-slave'
        label 'k8s-slave'
    }
    parameters {
        booleanParam(name: 'BUILD', defaultValue: true, description: "Run buid and push image")
        choice(name: 'TARGET_ENV', choices: ['dev', 'test', 'stage', 'prod'], description: 'Target environment for API url')
        booleanParam(name: 'SKIP_SONAR', defaultValue: false, description: 'Skip SonarQube analysis and Quality gate')
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

        // Kubernetes Dev Cluster Details 
        DEV_CLUSTER_NAME = "np-cluster"
        DEV_CLUSTER_ZONE = "us-east4-a"
        DEV_PROJECT_ID = "project-fe6816d0-c7fc-4c9b-bd7"
 
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
                    return params.BUILD && !params.SKIP_SONAR
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
                    return params.BUILD && !params.SKIP_SONAR
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
        stage ('GKE Auth') {
            when {
                expression {
                    return params.TARGET_ENV == 'dev'
                }
            }
            steps {
                script {
                    sh """
                        echo "******************************* Authenticating to GKE **************************"
                        gcloud container clusters get-credentials ${env.DEV_CLUSTER_NAME} --zone ${env.DEV_CLUSTER_ZONE} --project ${env.DEV_PROJECT_ID}

                        echo "******************** Validating the Cluster access *********************"
                        kubectl get nodes
                    """
                }
            }
        }
        stage ('DeployToDevEnvironment'){
            // GKE Cluster should be available - done
            // kubectl should be availble - done
            // slave should be having config file to connect to our api server - done 
            // Create k8s manifests file and make them apply into our namespaces - Pending 
            // Create a reusable code for all environments - pending
            when {
                expression {
                    return params.BUILD && params.TARGET_ENV == 'dev'
                }
            }
            steps {
                script {
                    env.NAMESPACE = 'i27-helpdesk-dev'
                    sh '''
                        echo "******************* Deploying to Dev Environment *********************"
                        echo "Deploying into this namespace: ${NAMESPACE}"
                        kubectl get pods -n ${NAMESPACE}
                        # Substitute variables in kubernetes manifests
                        sed -i "s|\\${NAMESPACE}|${NAMESPACE}|g" k8s/*.yaml
                        sed -i "s|\\${IMAGE_NAME}|${IMAGE_NAME}|g" k8s/deploy.yaml
                        sed -i "s|\\${IMAGE_TAG}|${GIT_COMMIT}|g" k8s/deploy.yaml
                        echo "Applying k8s manifests in dev namespace"
                        kubectl apply -f k8s/
                        echo "Deployment to Dev namespace is completed"

                    '''
                }
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


