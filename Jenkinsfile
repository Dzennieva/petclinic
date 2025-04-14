pipeline {
    agent any

tools{
    maven 'maven3.9'
}
 environment {
     SCANNER_HOME= tool 'sonar7.1'
 }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-cred', url: 'https://github.com/Dzennieva/petclinic.git']])
            }
        }
       
      stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
       
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('ibt-sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic'''
                }
            }
        }
          stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', nvdCredentialsId: 'nvd_key', odcInstallation: 'owasp'
            }
        }
        
        stage('Deploy to nexus') {
            steps {
              configFileProvider([configFile(fileId: 'maven-settings', variable: 'mavensettings')]) {
                    sh "mvn deploy -s $mavensettings -DskipTests=true"
              }
            }
        }

          stage('Deploy') {
            steps {
                sh 'sudo cp target/*.war /opt/apache-tomcat-9.0.65/webapps/'
            }
        }
    }
}
