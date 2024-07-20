pipeline {
    agent any
    tools {
        jdk "OracleJDK17"
        maven "MAVEN3"
    }
    environment {
          DOCKERIMAGE = "haleemo/complete-prodcution-e2e-pipeline"
          DOCKERPASS = "dockerhub"

        }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: '', url: 'https://github.com/haleem00/complete-prodcution-e2e-pipeline.git'
            }
        }

        stage("Build App") {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

//         stage('Sonarqube Analysis') {
//             steps {
//               script {
//                 withSonarQubeEnv(credentialsId: 'sonartoken') {
//                   sh 'mvn sonar:sonar'
//
//                 }
//               }
//             }
//         }

        stage('Sonar Scanner') {
          steps {
            script {
              withSonarQubeEnv(credentialsId: 'sonartoken2') {
                sh '''
                   mvn clean verify sonar:sonar \
                   -Dsonar.projectKey=product-key \
                   -Dsonar.projectName=product \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/main/java \
                   -Dsonar.tests=src/test/java \
                   -Dsonar.exclusions=src/test/java/**
                '''
              }
            }
          }
        }

         stage('Sonarqube QualityGate') {
            steps {
              script {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonartoken'
              }
            }
        }

         stage('Build and push docker image') {
                    steps {
                      script {
                        docker_image = docker.build("${DOCKERIMAGE}:V${BUILD_NUMBER}")
                        withDockerRegistry([url: '', credentialsId: DOCKERPASS]) {
                            docker_image.push("V${BUILD_NUMBER}")
                            docker_image.push("latest")
                           }
                       }
                   }
                }

        stage('Remove the unused docker image') {
                steps {
                  sh "docker system prune -af"
                }
            }

        stage('Trigger CD Pipeline') {
            steps {
                build job: 'cd-new'
            }
        }


    }
}
