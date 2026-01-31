pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
        DOCKER_IMAGE = "docharjeetsingh/my-app"
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout 'main' branch with credentials
                git branch: 'main', 
                    url: 'https://github.com/githarjeetsingh/ci-cd-project.git',
                    credentialsId: 'github-credential'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=ci-cd-jenkins-sonar-project \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://localhost:9000
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker stop my-app || true
                docker rm my-app || true
                docker run -d -p 80:8080 --name my-app $DOCKER_IMAGE
                """
            }
        }
    }
}
