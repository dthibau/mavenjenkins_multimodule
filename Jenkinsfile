pipeline {
  agent any
  tools {
      maven 'Maven3' 
  }
 stages {
    stage('Unit') {
      steps {
        sh './mvnw -Pprod clean package'
        stash includes: '**/target/*.jar', name: 'appli'
      }
    }
    
    
    stage('Parallel Stage') {
    
      parallel {
        stage('Integration tests') {
          agent any
          steps {  
            sh "mvn -Pintegration integration-test"
          }
        }
        stage('Qualité') {
          agent any
          steps {  
            withSonarQubeEnv('Local-SonarQube') {
                sh './mvnw clean verify sonar:sonar'
             }
          }
        }
      }
    }
    stage("Quality Gate") {
      steps {
          sleep(10)
          waitForQualityGate abortPipeline: true
        
       }
    }
    stage("Snapshot deployement") {
      steps {
          sh './mvnw -Pnexus deploy'
        
       }
    }
    stage('Validation métier') {
      agent none     
       input {
          message "Déploiement en prod ?"
          ok "Oui "
          parameters {
            string(name: 'DATA_CENTER', defaultValue: 'Paris', description: 'Data Center où déployer')
          }
        }
      steps {
        echo "Echo, let's go to prod"
      }
    }
    stage('Déploiement') {
     steps {
        unstash 'appli'  
        script {
          if ( params.DATA_CENTER == 'Paris') {
            sh 'cp application/target/*.jar /home/dthibau/Formations/MavenJenkins/MyWork/Serveur/Paris.jar'
          } else {
            sh 'cp application/target/*.jar /home/dthibau/Formations/MavenJenkins/MyWork/Serveur/Bruxelles.jar'
          }
        }
     }
    } 
 }           
}
 
