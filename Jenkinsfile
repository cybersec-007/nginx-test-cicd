pipeline {
    agent any

    environment {
        IMAGE_NAME = "nginx-test"
        CONTAINER_NAME = "nginx-test-container"
        HOST_PORT = "9889"
        CONTAINER_PORT = "80"
        FILE_PATH = "index.html"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: """
                    FROM nginx:alpine
                    COPY ${FILE_PATH} /usr/share/nginx/html/index.html
                    """
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d --name $CONTAINER_NAME -p $HOST_PORT:$CONTAINER_PORT $IMAGE_NAME'
                sh 'sleep 5' // время на старт
            }
        }

        stage('Check HTTP Response') {
            steps {
                sh '''
                    CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:$HOST_PORT)
                    if [ "$CODE" != "200" ]; then
                        echo "ERROR: HTTP code $CODE"
                        exit 1
                    fi
                '''
            }
        }

        stage('Check MD5') {
            steps {
                sh '''
                    ORIG=$(md5sum $FILE_PATH | awk \'{ print $1 }\')
                    REMOTE=$(curl -s http://localhost:$HOST_PORT | md5sum | awk \'{ print $1 }\')
                    echo "Original: $ORIG"
                    echo "Returned: $REMOTE"
                    [ "$ORIG" = "$REMOTE" ] || exit 1
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f $CONTAINER_NAME || true'
        }
        failure {
            script {
                // Пример Telegram-уведомления, если установлен telegram-send:
                sh 'telegram-send "Ошибка в пайплайне Jenkins: $BUILD_URL"'
            }
        }
    }
}
