pipeline {
  agent any
  stages {
    stage('Static Code Analysis') {
      agent any
      steps {
        sh '''current_time=$(date "+%Y-%m-%d-%H-%M-%S")
echo "Current Time : $current_time"

mkdir /app/bin/jenkins_home/workspace/application-code_master/results

cd /app/cloned_repos/microsoft-application-inspector/ApplicationInspector
dotnet ApplicationInspector.CLI.dll analyze -s /app/cloned_repos/application-code -f json  -o /app/bin/jenkins_home/workspace/application-code_master/results/results.json

jq \'.metaData.detailedMatchList[].severity\' /app/bin/jenkins_home/workspace/application-code_master/results/results.json > results.txt

ls -al

File=results.txt  
if grep -q Critical "$File"; ##note the space after the string you are searching for
then
   echo "Critical issue caught"
   #exit 1
else
   echo "No critical issues found!"
fi'''
      }
    }

    stage('Deploy to QA Server') {
      steps {
        echo 'Deploying to QA Server'
      }
    }

    stage('Regression Testing') {
      parallel {
        stage('API Tests') {
          steps {
            echo 'Running JMeter API Tests'
            sh '''pwd
cd /app/cloned_repos/pc-portal-api-tests'''
            sh '''cd /app/cloned_repos/pc-portal-api-tests
jmeter -Jjmeter.save.saveservice.output_format=xml -n -t "PC Portal API Test Plan.jmx" -l /app/cloned_repos/pc-portal-api-tests/pc-portal-api-test-plan.jtl'''
            perfReport '/app/cloned_repos/pc-portal-api-tests/pc-portal-api-test-plan.jtl'
          }
        }

        stage('UI Tests') {
          agent any
          steps {
            echo 'Running P8 UI Tests'
            sh '''curl --insecure --header "Content-Type: application/json" --request POST --data \'{ "test_run_id": 282, "os": "win10", "browser": "chrome", "browser_version": "70.0.3538.0", "from_url": "", "execution_platform_type": "web", "project_id": 425 }\' https://platform.fogchaininc.com:7300/v1/test_run/execute_test_run_sync?key=ZYwAlL51RRJx3XCg
pwd
wget --no-check-certificate --content-disposition "https://dev.prometheus8.com/report_status?type=complete&company_key=ZYwAlL51RRJx3XCg&test_run_id=282"
ls -al
mv 425-Onboarding-web-selenium_grid-win10-chrome.zip /app/bin/jenkins_home/workspace/application-code_master
curl --insecure --fail --header "Content-Type: application/json" "https://platform.fogchaininc.com/report_status?type=has_failures&company_key=ZYwAlL51RRJx3XCg&test_run_id=282"'''
          }
        }

      }
    }

    stage('Performance Testing') {
      steps {
        echo 'Running JMeter Load Tests'
        sh '''pwd
cd /app/cloned_repos/pc-portal-api-tests'''
        sh '''cd /app/cloned_repos/pc-portal-api-tests
ls -al
jmeter -Jjmeter.save.saveservice.output_format=xml -n -t "PC Portal Load Test Plan.jmx" -l /app/cloned_repos/pc-portal-api-tests/pc-portal-load-test-plan.jtl'''
        perfReport '/app/cloned_repos/pc-portal-api-tests/pc-portal-load-test-plan.jtl'
      }
    }

    stage('Deploy to Production') {
      steps {
        echo 'Pipeline passed! Deploying to Production Server'
        sh '''pwd
ls -al'''
      }
    }

  }
  post {
    always {
      archiveArtifacts(artifacts: 'results/*.json', onlyIfSuccessful: false)
      archiveArtifacts(artifacts: '425-Onboarding-web-selenium_grid-win10-chrome.zip', onlyIfSuccessful: false)
    }

  }
}