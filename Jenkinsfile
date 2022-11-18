def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools{
        maven 'localMaven'
        jdk 'localJdk'
    }
    
    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/Vstar007/Jenkins-CICD-Pipeline-Project.git'
            }
        }
        
        
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'archiving....'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
    stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
    }
    stage('Integration Test'){
        steps {
          sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('Checkstyle Code Analysis'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    stage ('SonarQube scanning'){
        steps {
            withSonarQubeEnv('SonarQube') {
            sh """
          mvn sonar:sonar \
  -Dsonar.projectKey=JavaWebApp \
  -Dsonar.host.url=http://172.31.38.61:9000 \
  -Dsonar.login=0d50552db6557fd5c6e0cff779171ba0531a8c4d
            """
            }
        }
    }
  stage("Quality Gate"){
      steps{
       waitForQualityGate abortPipeline: true
         }
    }


    stage("Upload artifact to Nexus"){
      steps{
          
        sh 'mvn clean deploy -DskipTests'
         }
    }
    
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
    stage('Deploy to Stage') {
      environment {
        HOSTS = "stage"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
    stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    }
     
    stage('Deploy to Prod') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
    }
    post {
    always {
        echo 'Slack Notifications.'
        slackSend channel: '#vstar-cicd-alerts',
        color: COLOR_MAP[currentBuild.currentResult],
        message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
    }
  }
    
}