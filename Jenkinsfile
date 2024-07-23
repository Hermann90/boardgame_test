pipeline {    
    agent any 
    tools{
        maven 'M2_HOME'
    }

    // Define the global env variable 
    environment {
        SCANNER_HOME= tool 'sonar'
        ARTIFACTORY_URL = 'http://52.10.149.220:8081/artifactory/geolocation/'
        ARTIFACTORY_REPO = 'geolocation'
        RELEASE_VERSION = 'jun-24-v2'
        
        def mavenPom = readMavenPom file: 'pom.xml'
        POM_VERSION = "${mavenPom.version}"
        APP_NAME = "${mavenPom.name}"
        //def pom = readMavenPom file: 'pom.xml'
    }


    stages {   
        stage('git checkout') {
            steps {
               git branch: 'main', credentialsId: 'HERMANN90-GITHUB-TOKEN', \
               url: 'https://github.com/Hermann90/boardgame_test.git'
            }
        }

        stage('Compile'){
            steps{
                sh "mvn clean"
                sh "mvn compile"
            }
        }
/*        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Trivy Scan File') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=boardgameTest -Dsonar.projectKey=boardgameTest \
                            -Dsonar.java.binaries=. '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'SONAR-TOKEN' 
                }
            }
        }
*/
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }

        stage('Upload Jar to Jfrog'){
            
            //def appName = pom.app
            steps{
                 withCredentials([usernamePassword(credentialsId: 'artifact-jgrogID', \
                 usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    script {

                        
                        echo "${POM_VERSION}"
                        echo "${APP_NAME}"
                        // Define the artifact path and target location
                        def artifactPath = 'target/*.jar'
                        def targetPath = "${ARTIFACTORY_REPO}/${APP_NAME}-${POM_VERSION}.jar"

                        // Upload the artifact using curl
                        //sh "echo ${pom.name}"
                        sh """
                            curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD} \
                                -T ${artifactPath} \
                                ${ARTIFACTORY_URL}/${targetPath}
                        """
                        }
                    }
                }

        }

        stage('Build Docker Image'){
            steps{
                script{
                    sh """docker build -t hermann90/${APP_NAME}:latest .""" 
                    sh """docker build -t hermann90/${APP_NAME}:${POM_VERSION} ."""
                }
            }
        }

        stage('Scan Image'){
            steps{
                sh   """trivy image --format table -o docker_image_report.html hermann90/${APP_NAME}:${POM_VERSION}"""
            }
        }

        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'registryID', toolName: 'docker') {
                        sh """docker push hermann90/${APP_NAME}:latest"""
                        sh """docker push hermann90/${APP_NAME}:${POM_VERSION}"""
                        sh """docker rmi -f hermann90/${APP_NAME}:latest hermann90/${APP_NAME}:${POM_VERSION}"""
                    }
               }
            }
        }

        stage('Build Helm With Value') {
            steps {
               script {
                    sh """cat app_deploy/values.yaml"""
                    sh """yq -i '.image.tag = "${POM_VERSION}"' app_deploy/values.yaml"""
                    sh """yq -i '.image.repository = "hermann90/${APP_NAME}"' app_deploy/values.yaml"""
                    sh """cat app_deploy/values.yaml"""
               }
            }
        }

    }
}
