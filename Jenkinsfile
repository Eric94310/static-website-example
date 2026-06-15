pipeline {

    environment {
        IMAGE_NAME = "static-website-example"
        IMAGE_TAG = "latest"
        STAGING = "eric-static-staging"
        PRODUCTION = "eric-static-prod"
    }

    agent none

    stages {

        stage('Checkout Source Code') {
            agent any

            steps {
                git branch: 'master',
                    url: 'https://github.com/Eric94310/static-website-example.git'
            }
        }

        stage('Build Docker Image') {
            agent any

            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Run Container') {
            agent any

            steps {
                sh '''
                    docker rm -f $IMAGE_NAME || true
                    docker run --name $IMAGE_NAME -d -p 8084:80 $IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                '''
            }
        }

        stage('Test Application') {
            agent any

            steps {
                sh 'curl -f http://172.17.0.1:8084'
            }
        }

        stage('Clean Test Container') {
            agent any

            steps {
                sh '''
                    docker stop $IMAGE_NAME || true
                    docker rm $IMAGE_NAME || true
                '''
            }
        }

        stage('Package Docker Image') {
            agent any

            steps {
                sh 'docker image inspect $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Deploy to Staging') {
            when {
                expression {
                    env.GIT_BRANCH == 'origin/master' || env.GIT_BRANCH == 'master'
                }
            }

            agent any

            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }

            steps {
                sh '''
                    heroku container:login
                    heroku create $STAGING || echo "Application staging déjà créée"
                    heroku stack:set container -a $STAGING
                    heroku container:push web -a $STAGING
                    heroku container:release web -a $STAGING
                '''
            }
        }

        stage('Manual Validation') {
            agent any

            steps {
                input message: 'Valider le déploiement en production ?'
            }
        }

        stage('Deploy to Production') {
            when {
                expression {
                    env.GIT_BRANCH == 'origin/master' || env.GIT_BRANCH == 'master'
                }
            }

            agent any

            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }

            steps {
                sh '''
                    heroku container:login
                    heroku create $PRODUCTION || echo "Application production déjà créée"
                    heroku stack:set container -a $PRODUCTION
                    heroku container:push web -a $PRODUCTION
                    heroku container:release web -a $PRODUCTION
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline exécutée avec succès'
        }

        failure {
            echo 'La pipeline a échoué'
        }
    }
}
