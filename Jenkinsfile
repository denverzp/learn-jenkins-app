pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "Test stage"
                    if (fileExists('build/index.html')){
                        sh '''
                            test -f build/index.html
                        '''
                        echo "Ok! File build/index.html exists"
                    } else {
                        echo "Error! File build/index.html not exists"
                    }
                }
            }
        }
    }
}
