pipeline {
    agent {
        label 'Slave'
    }
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "m3"
    }
    environment{
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    } 
    stages {
        stage('Clear running apps') {
           steps {
               // Clean previous instances of app built
               sh 'docker rm -f pandaapp || true'
           }
        }
        stage('Get Code') {
            steps {
                // Get some code from a GitHub repository.
                checkout scm
                //git branch: 'pipeline', url: 'https://github.com/nygamichal/panda_applicationALL.git'
            }
        }
        stage('Build and Junit') {
            steps {
                // Run Maven on a Unix agent.
                sh 'mvn clean install'
            }
        }
        stage('Build Docker image'){
            steps {
                sh 'mvn package -Pdocker'
                //sh 'mvn package -Pdocker -Dmaven.test.skip=true'
            }
        }
        stage('Run Docker app') {
            steps {
                sh 'docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}'
            }
        }
        stage('Test Selenium') {
            steps {
                sh 'mvn test -Pselenium'
            }
        }
        stage('Deploy jar to artifactory') {
            steps {
                configFileProvider([configFile(fileId: '73db171a-2bf5-442b-9e85-0ca67787a243', variable: 'mavensettings')]) {
                    sh 'mvn -s $mavensettings deploy -Dmaven.test.skip=true -e'
                }
            } 
            post {
                always { 
                    sh 'docker stop pandaapp'
                    deleteDir() //Clean workspace
                }
            }
        }
    }
}
