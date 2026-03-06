pipeline {
    agent any

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
                    def app = docker.build("mohammed123/petclinic-spinnaker-jenkins")
                    env.APP_IMAGE = "mohammed123/petclinic-spinnaker-jenkins"
                }
            }
        }

        stage('Push Docker Image') {
            when { branch 'master' }
            steps {
                script {
                    def GIT_COMMIT_HASH = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    def SHORT_COMMIT = GIT_COMMIT_HASH.take(8)

                    docker.withRegistry('https://registry-1.docker.io', 'dockerHubCredentials') {
                        def app = docker.image(env.APP_IMAGE)
                        app.push(SHORT_COMMIT)
                        app.push("latest")
                    }

                    env.SHORT_COMMIT = SHORT_COMMIT
                }
            }
        }

        stage('Remove local images') {
            when { branch 'master' }
            steps {
                sh "docker rmi -f mohammed123/petclinic-spinnaker-jenkins:latest || true"
                sh "docker rmi -f mohammed123/petclinic-spinnaker-jenkins:${env.SHORT_COMMIT} || true"
            }
        }
    }
}
