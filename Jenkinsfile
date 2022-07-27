pipeline {
  agent any

  tools {
    maven "Maven"
  }

  environment {
    NEXUS_VERSION = "nexus3"
    NEXUS_PROTOCOL = "http"
    NEXUS_URL = "10.107.8.63:8081"
    NEXUS_REPOSITORY = "maven-nexus-repo"
    NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
    DOCKERHUB_CREDENTIALS=credentials('dockerhub')
  }

  stages {
    stage('Clone the Git') {
        steps {
            git 'https://github.com/Oussama-Goumghar/java_web_app.git'
        }
        
    }    
    stage('SonarQube analysis') {
        steps {
            script {
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
    stage('Quality Gates'){
        steps {
            timeout(time: 2, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
       }   
    }

    stage("Maven Build") {
        steps {
            script {
                sh "mvn package -DskipTests=true"
            }
        }
    }

    stage("Publish to Nexus Repository Manager") {
        steps {
            script {
                pom = readMavenPom file: "pom.xml";
                filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                artifactPath = filesByGlob[0].path;
                artifactExists = fileExists artifactPath;
                if(artifactExists) {
                    echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging],
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: "pom.xml",
                            type: "pom"]
                        ]
                    );
                } else {
                    error "*** File: ${artifactPath}, could not be found";
                }
            }
        }
    }

    stage('Build') {
        steps {
            script {
                sh 'docker build -t thetips4you/nodeapp_test:latest .'
            }
        }
	}


  }
}
