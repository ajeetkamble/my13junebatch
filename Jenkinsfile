pipeline {
    agent any

    environment {
        // Replace '/' in JOB_NAME (folders / multibranch) so it is a valid Docker tag
        SAFE_JOB    = "${env.JOB_NAME.replace('/', '-')}"
        IMAGE_REPO  = 'ajeetkamble/tgdevops13june'
        IMAGE_TAG   = "${SAFE_JOB}-${env.BUILD_NUMBER}"
        IMAGE_FULL  = "${IMAGE_REPO}:${IMAGE_TAG}"
        NAMESPACE   = 'mytg'
        // Jenkins credentials ID (Username with password: Docker Hub username + PAT)
        DOCKER_CREDS_ID = 'dockerhub-pat'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Start') {
            steps {
                echo 'We are starting project'
                echo "Your project name is (Jobname) and build number is--> ${SAFE_JOB}-${BUILD_NUMBER}"
                sh 'ls -ltr'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_FULL} --no-cache ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDS_ID,
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PAT')]) {
                    sh '''
                        echo "$DOCKER_PAT" | docker login -u "$DOCKER_USER" --password-stdin
                        docker image push "$IMAGE_FULL"
                        docker logout
                    '''
                }
            }
        }

        stage('Prepare Manifest') {
            steps {
                sh '''
                    sed -i "s/pname/${SAFE_JOB}/g" tomcat-deploy.yaml
                    sed -i "s/buildnum/${BUILD_NUMBER}/g" tomcat-deploy.yaml
                    cat tomcat-deploy.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl get all
                    kubectl get svc -n ${NAMESPACE}
                    kubectl apply -f mynps.yaml
                    kubectl apply -f tomcat-deploy.yaml
                    kubectl get all -n ${NAMESPACE}
                    kubectl get svc -n ${NAMESPACE}
                '''
            }
        }

        stage('Access Info') {
            steps {
                sh '''
                    echo "You can check latest build working on following location"
                    curl -s ifconfig.io
                    echo
                    echo "Check for example ---> http://<publicIP>:8080"
                '''
            }
        }
    }

    post {
        success {
            echo "Build ${BUILD_NUMBER} deployed successfully: ${IMAGE_FULL}"
        }
        failure {
            echo "Build ${BUILD_NUMBER} failed. Check the stage logs above."
        }
        always {
            // Remove the local image so the agent disk doesn't fill up
            sh "docker rmi ${IMAGE_FULL} || true"
        }
    }
}
