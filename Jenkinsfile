pipeline {
    agent none 
    environment {
        docker_app = "go_app"
        GOCACHE = "/tmp"
        registry = "foo"
        userid = """${sh(
                returnStdout: true,
                script: 'echo "$USER"'
            )}"""
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
                    sh 'docker login -u admin -p registry https://${registry}:443'
                    sh 'docker build -t ${registry}:443/go_app:$BUILD_NUMBER .'
                    sh 'docker push ${registry}:443/go_app:$BUILD_NUMBER'
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
                    sh "sed -i 's/REGISTRY/${registry}/g' deployment.yml"
                    sh "sed -i 's/DOCKER_APP/${docker_app}/g' deployment.yml"
                    sh "sed -i 's/BUILD_NUMBER/${BUILD_NUMBER}/g' deployment.yml"
                    sh 'scp -r -v -o StrictHostKeyChecking=no *.yml ${userid}@${registry}:~/'
                    sh 'ssh -o StrictHostKeyChecking=no ${userid}@${registry} kubectl apply -f /users/${userid}/deployment.yml'
                    sh 'ssh -o StrictHostKeyChecking=no ${userid}@${registry} kubectl apply -f /users/${userid}/service.yml'                                        
                }
            }
        }
    }
}
