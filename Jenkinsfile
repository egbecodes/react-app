pipeline {
    agent any
    stages {
        stage('Build React App') {
            steps {
                echo 'Running build automation'
                sh 'chmod +x gradlew'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/reactApp'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    app = docker.build("oashu/react-app")
                    app.inside {
                        sh 'echo $(curl localhost:1233)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                // echo '$DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                     docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_login') {
                    // echo 'docker tag my-image oashu/apps_repo:my-tag'
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                     }
                }
            }
        }
         stage('DeployToStaging') {
            when {
                branch 'main'
            }
            steps {
                    script {
                        sh "docker pull oashu/react-app:${env.BUILD_NUMBER}"
                        try {
                            sh "docker stop react-app"
                            sh "docker rm react-app"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "docker run --restart always --name react-app -p 1233:80 -d oashu/react-app:${env.BUILD_NUMBER}"
                    }
            }
        }
        
        stage("Check HTTP Response") {
            steps {
                script {
                    final String url = "http://localhost:1233"
                    
                    final String response = sh(script: "curl -o /dev/null -s -w '%{http_code}\\n' $url", returnStdout: true).trim()
                    
                    if (response == "200") {
                        echo response
                        println "Successful Response Code" 
                    } else {
                        echo response
                        println "Error Response Code" 
                    }

                }
            }
        }
        
        stage('DeployToProduction') {
            when {
                branch 'main'
            }
            steps {
                input 'Does the staging environment look OK? Did You get 200 response?'
                 milestone(1)
                    script {
                        sh "docker pull oashu/react-app:${env.BUILD_NUMBER}"
                        try {
                            sh "docker stop react-app"
                            sh "docker rm react-app"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "docker run --restart always --name react-app -p 1233:80 -d oashu/react-app:${env.BUILD_NUMBER}"
                    }
            }
        }
    }
}
