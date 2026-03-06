pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'mominuddin0573'   // <-- change only this if needed
        IMAGE_NAME     = 'petclinic-spinnaker-jenkins'
        APP_IMAGE      = "${DOCKERHUB_USER}/${IMAGE_NAME}"
    }

    stages {
        stage('Build Application') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test Application') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            when { branch 'master' }
            steps {
                script {
                    docker.build(env.APP_IMAGE)
                }
            }
        }

        stage('Push Docker Image') {
            when { branch 'master' }
            steps {
                script {
                    def GIT_COMMIT_HASH = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    env.SHORT_COMMIT = GIT_COMMIT_HASH.take(8)

                    docker.withRegistry('https://registry-1.docker.io', 'dockerHubCredentials') {
                        def app = docker.image(env.APP_IMAGE)
                        app.push(env.SHORT_COMMIT)
                        app.push("latest")
                    }
                }
            }
        }

        stage('Remove local images') {
            when { branch 'master' }
            steps {
                sh "docker rmi -f ${env.APP_IMAGE}:latest || true"
                sh "docker rmi -f ${env.APP_IMAGE}:${env.SHORT_COMMIT} || true"
            }
        }
    }
}
