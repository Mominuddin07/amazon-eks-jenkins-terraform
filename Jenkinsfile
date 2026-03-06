pipeline {
    agent any

    triggers {
        pollSCM "* * * * *"
    }

    stages {

        stage('Build Application') { 
            steps {
                echo '=== Building Petclinic Application ==='
                sh 'mvn -B -DskipTests clean package' 
            }
        }

        stage('Test Application') {
            steps {
                echo '=== Testing Petclinic Application ==='
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
                echo '=== Building Petclinic Docker Image ==='
                script {
                    def app = docker.build("mohammed123/petclinic-spinnaker-jenkins")
                }
            }
        }

        stage('Push Docker Image') {
            when { branch 'master' }
            steps {
                echo '=== Pushing Petclinic Docker Image ==='
                script {
                    def app = docker.image("mohammed123/petclinic-spinnaker-jenkins")

                    def GIT_COMMIT_HASH = sh(
                        script: "git rev-parse HEAD",
                        returnStdout: true
                    ).trim()

                    def SHORT_COMMIT = GIT_COMMIT_HASH.take(8)

                    docker.withRegistry('https://index.docker.io/v1/', 'dockerHubCredentials') {
                        app.push(SHORT_COMMIT)
                        app.push("latest")
                    }
                }
            }
        }

        stage('Remove local images') {
            steps {
                echo '=== Delete the local docker images ==='
                sh "docker rmi -f mohammed123/petclinic-spinnaker-jenkins:latest || true"
                sh "docker rmi -f mohammed123/petclinic-spinnaker-jenkins:* || true"
            }
        }
    }
}
