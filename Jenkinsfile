pipeline {
  agent any

  stages {
    stage('Clone the Git') {
        steps {
            git 'https://github.com/Oussama-Goumghar/maven-basic-project.git'
        }
        
    }    
    stage('SonarQube analysis') {
        steps {
            def scannerHome = tool 'sonarqube';
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner \
                -D sonar.login=admin \
                -D sonar.password=sonar \
                -D sonar.projectKey=project \
                -D sonar.exclusions=vendor/**,resources/**,**/*.java \
                -D sonar.host.url=http://10.109.227.155/"
            }
        } 
    }  
  }
}
