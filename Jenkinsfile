pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_USER = 'dhruvakssample'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube') {
    steps {
        sh '''
        mvn sonar:sonar \
        -Dsonar.projectKey=springboot-devops \
        -Dsonar.host.url=http://172.17.0.1:9000 \
        -Dsonar.token=$SONAR_TOKEN
        '''
    }
}
        
        stage('Upload To Nexus') {
    steps {
        sh '''
        mvn deploy
        '''
    }
}
        stage('Docker Build') {
            steps {
                sh '''
                docker build -t hello-app:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                    docker tag hello-app:${BUILD_NUMBER} dhruvakssample/hello-app:${BUILD_NUMBER}

                    docker push dhruvakssample/hello-app:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop hello-app || true

                docker rm hello-app || true

                docker pull dhruvakssample/hello-app:${BUILD_NUMBER}

                docker run -d \
                  --name hello-app \
                  -p 8082:8082 \
                  dhruvakssample/hello-app:${BUILD_NUMBER}
                '''
            }
        }
    }
}
