pipeline {
    agent any 
    
    tools{
        jdk 'jdk_9'
        maven 'maven_3.9'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage("clean workspace"){
            steps{
                cleanWs()
            }
        }
    }
    stages {
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/vishnutheja88/new_petclinic.git'
            }
        }
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
                }
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
         stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        
                        sh "docker build --build-arg TMDB V3 API_KEY=dckr_pat_-ahjoa0H1SrPDwsgURdWBS7ucPE image1 ."
                        sh "docker tag image1 vsihnutheja88/petclinic-test:latest "
                        sh "docker push vsihnutheja88/petclinic-test:latest "
                    }
                }
            }
        }
        
        stage("TRIVY"){
            steps{
                sh " trivy image vsihnutheja88/petclinic-test:latest"
            }
        }
        
        stage("Deploy To Tomcat"){
            steps{
                sh "cp  /var/lib/jenkins/workspace/CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        }
    }
}
