pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            inheritFrom 'test-1-agent'
            serviceAccount 'test-1-sa' 
            yaml '''
                kind: Pod
                metadata:
                  namespace: test-1-namespace
                spec:
                  serviceAccountName: test-1-sa
                  containers:
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:debug
                    command:
                    - /busybox/cat
                    tty: true
                    volumeMounts:
                    - name: docker-config
                      mountPath: /kaniko/.docker/
                  - name: kubectl
                    image: bitnami/kubectl:latest
                    command:
                    - sleep
                    args:
                    - 99d
                    tty: true
                    securityContext:
                      runAsUser: 1000
                  volumes:
                  - name: docker-config
                    secret:
                      secretName: docker-credentials
                      items:
                        - key: .dockerconfigjson
                          path: config.json
            '''
            defaultContainer 'kubectl'
            namespace 'test-1-namespace'
        }
    }
    
    environment {
        DOCKER_IMAGE = "taeyoondev/jenkins-test"
        DOCKER_TAG = "${BUILD_NUMBER}"
        GIT_REPO = "https://github.com/gogoyooni/jenkins-service-test.git"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: "${GIT_REPO}"
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                            /kaniko/executor \
                            --context=${WORKSPACE} \
                            --dockerfile=${WORKSPACE}/Dockerfile \
                            --destination=${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    script {
                        sh """
                            echo "Starting deployment..."
                            sed -i 's|\${DOCKER_IMAGE}|${DOCKER_IMAGE}|g' k8s/deployment.yaml
                            sed -i 's|\${DOCKER_TAG}|${DOCKER_TAG}|g' k8s/deployment.yaml
                            kubectl apply -f k8s/deployment.yaml -n test-1-namespace
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
