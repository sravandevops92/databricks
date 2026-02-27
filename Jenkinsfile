pipeline {
  agent any 
    stages {
      stage ('sonar analysis') {
        steps {
          script {
            println("${env.BRANCH_NAME}")
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
            rm -f *.zip
            '''
          }
        }
      }
      stage ('deploy'){
        steps {
          script {
            def tokenId = ''
            def host_url = ''
            def environment_name = ''

            if (env.BRANCH_NAME == 'develop') {
                tokenId = 'databrickstoken'
                host_url = 'https://dbc-cadbc0fd-96cb.cloud.databricks.com'
                environment_name = 'dev'
            } 
            else if (env.BRANCH_NAME == 'qa') {
                tokenId = 'databrickstoken'
                host_url = 'https://dbc-cadbc0fd-96cb.cloud.databricks.com'
                environment_name = 'dev'
            } 
            else if (env.BRANCH_NAME == 'main') {
                tokenId = 'databrickstoken'
                host_url = 'https://dbc-cadbc0fd-96cb.cloud.databricks.com'
                environment_name = 'dev'
            } 
            else {
                error "No token configured for branch ${env.BRANCH_NAME}"
            }

            withCredentials([string(credentialsId: tokenId, variable: 'TOKEN')]) {
                println("######### GENERING DATABRICKS CONFIG FILE #########")
                env.DATABRICKS_TOKEN = TOKEN
                /*sh """
                 touch ~/.databrickscfg
                 echo "[DEFAULT]" >> ~/.databrickscfg
                 echo "token = ${TOKEN} >> ~/.databrickscfg
                 echo "host = ${host_url} >> ~/.databrickscfg
                 cat  ~/.databrickscfg
                """ */
                println("############ DATABRICKS BUNDLE VALIDATE #########")
                sh "databricks bundle validate -t ${environment_name}"
                println("############ DATABRICKS BUNDLE DEPLOY #########")
                sh "databricks bundle deploy -t ${environment_name} --force-lock"
            }
          }
        }
      }
    }
}
