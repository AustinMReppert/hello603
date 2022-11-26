pipeline {
    agent none 
    environment {
        docker_app = "go_app"
        GOCACHE = "/tmp"
    }
    stages {
        stage('Build') {
            agent {
                kubernetes {
                    inheritFrom 'golang'
                }
            }
            steps {
                container('golang') {
                    // Create our project directory.
                    sh 'cd ${GOPATH}/src'
                    sh 'mkdir -p ${GOPATH}/src/hello-world'
                    // Copy all files in our Jenkins workspace to our project directory.                
                    sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
                    // Build the app.
                    sh 'export GO111MODULE=auto; go build'  
                }
            }     
        }
        stage('Test') {
            agent {
                kubernetes {
                    inheritFrom 'golang'
                }
            }
            steps {
                container('golang') {                 
                    // Create our project directory.
                    sh 'cd ${GOPATH}/src'
                    sh 'mkdir -p ${GOPATH}/src/hello-world'
                    // Copy all files in our Jenkins workspace to our project directory.                
                    sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
                    // Remove cached test results.
                    sh 'go clean -cache'
                    // Run Unit Tests.
                    sh 'export GO111MODULE=auto; go test ./... -v -short'            
                }
            }
        }
        stage('Publish') {
            agent {
                kubernetes {
                    inheritFrom 'docker'
                }
            }
            steps{
                container('docker') {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASSWORD $DOCKER_REGISTRY'
                    sh 'docker build -t $(echo $DOCKER_REGISTRY)/go_app:$BUILD_NUMBER .'
                    sh 'docker push $(echo $DOCKER_REGISTRY)/go_app:$BUILD_NUMBER'
                }
            }
        }
        stage ('Deploy') {
            agent {
                node {
                    label 'deploy'
                }
            }
            steps {
                sshagent(credentials: ['cloudlab']) {
                    sh "sed -i 's@REGISTRY@$DOCKER_REGISTRY@g' deployment.yml"
                    sh "sed -i 's/DOCKER_APP/${docker_app}/g' deployment.yml"
                    sh "sed -i 's@BUILD_NUMBER@${BUILD_NUMBER}@g' deployment.yml"
                    sh 'scp -r -v -o StrictHostKeyChecking=no *.yml $CLUSTER_USER@$CLUSTER_HOST:~/'
                    sh 'ssh -o StrictHostKeyChecking=no $CLUSTER_USER@$CLUSTER_HOST kubectl apply -f /users/$CLUSTER_USER/deployment.yml'                                    
                }
            }
        }
    }
}
