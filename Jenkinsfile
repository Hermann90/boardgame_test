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

                        def mavenPom = readMavenPom file: 'pom.xml'
                        POM_VERSION = "${mavenPom.version}"
                        APP_NAME = "${mavenPom.name}"
                        echo "${POM_VERSION}"
                        echo "${APP_NAME}"
                        // Define the artifact path and target location
                        def artifactPath = 'target/*.jar'
                        def targetPath = "${ARTIFACTORY_REPO}/release-${BUILD_ID}.jar"

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
                    sh 'docker build -t kserge2001/geo-app-img:latest .' 
                    sh 'docker build -t kserge2001/geo-app-img:${BUILD_ID} .'
                }
            }
        }

        stage('Scan Image'){
            steps{
                sh   'trivy image --format table -o docker_image_report.html kserge2001/geo-app-img:latest'
            }
        }


    }
}
