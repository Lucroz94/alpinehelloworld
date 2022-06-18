pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "lucroz94-staging"
        PRODUCTION = "lucroz94-production"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t lucroz94/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Run container based on build image') {
            agent any
            steps {
                script {
                    sh '''
                    docker run --name ${IMAGE_NAME} -d -p 80:5000 -e PORT=5000 lucroz94/${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 5s
                    '''

                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                    curl http://172.17.0.1 | grep -q "Hello World!"
                    '''
                }
            }
        }
        stage('Clean container') {
            agent any
            steps {
                script {
                    sh '''
                    docker stop ${IMAGE_NAME}
                    docker rm ${IMAGE_NAME}
                    '''
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                    heroku container:login
                    heroku create $STAGING || echo "project already exist"
                    heroku container:push -a $STAGING web
                    heroku container:release -a $STAGING web
                    '''
                }
            }
        }
        stage('Push image in production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                    heroku container:login
                    heroku create $PRODUCTION || echo "project already exist"
                    heroku container:push -a $PRODUCTION web
                    heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }
    }
}
