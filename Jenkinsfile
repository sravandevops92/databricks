pipeline {
  agent any 
    stages {
      stage ('sonar analysis') {
        steps {
          script {
            def SONAR_SCANNER = tool name: 'sonar-scanner'
            withSonarQubeEnv(credentialsId: 'sonartoken') {
              sh "${SONAR_SCANNER}/bin/sonar-scanner -Dsonar.projectKey=databricks -Dsonar.language=python"
           }
          }
        }
      }
      stage ('package') {
        steps {
          script {
            sh '''
            DATE=$(date +%Y-%m-%d_%H-%M-%S)
            zip -r application_${DATE}.zip .
            ls -la
            '''
          }
        }
      }
      stage ('deploy'){
        steps {
          script {
            def tokenId = ''
            def host_url = ''
            def env = ''

            if (env.BRANCH_NAME == 'develop') {
                tokenId = 'dev-databricks-token'
                host_url = 'https://dbc-cadbc0fd-96cb.cloud.databricks.com'
                env = 'dev'
            } 
            else if (env.BRANCH_NAME == 'qa') {
                tokenId = 'dev-databricks-token'
                host_url = 'https://dbc-cadbc0fd-96cb.cloud.databricks.com'
                env = 'dev'
            } 
            else if (env.BRANCH_NAME == 'main') {
                tokenId = 'dev-databricks-token'
                host_url = 'https://dbc-cadbc0fd-96cb.cloud.databricks.com'
                env = 'dev'
            } 
            else {
                error "No token configured for branch ${env.BRANCH_NAME}"
            }

            withCredentials([string(credentialsId: tokenId, variable: 'TOKEN')]) {
                println("######### GENERING DATABRICKS CONFIG FILE #########")
                sh """
                 touch ~/.databrickscfg
                 echo "[DEFAULT]" >> ~/.databrickscfg
                 echo "token = ${TOKEN} >> ~/.databrickscfg
                 echo "host = ${host_url} >> ~/.databrickscfg
                 cat  ~/.databrickscfg
                """
                println("############ DATABRICKS BUNDLE VALIDATE #########")
                sh "databricks bundle validate -t ${env}"
                println("############ DATABRICKS BUNDLE DEPLOY #########")
                sh "databricks bundle deploy -t ${env} --force-lock"
            }
          }
        }
      }
    }
}
