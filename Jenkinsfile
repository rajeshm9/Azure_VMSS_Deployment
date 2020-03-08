pipeline {
    agent any
    environment {
		RELEASE_DATE = new Date().format('yy.M.d')
		BUILD_NUM = "${RELEASE_DATE}.${BUILD_TAG}"
		RESOURCE_GRP="RESOURCE_GRP"
		VMSS_NAME="VMSS_NAME"
		COMMIT_INFO =sh(script: "git shortlog -1 -U ${env.GIT_COMMIT}", returnStdout: true).trim()
    }

    options {
       
        timeout(time: 30, unit: "MINUTES")
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '25'))

    }
    triggers {
	GenericTrigger(
    	 genericVariables: [
	      	[key: 'branch', value: '$.push.changes[0].new.name']

     	],

     	causeString: 'Triggered on $branch Branch ',

     	token: env.JOB_NAME,

	printContributedVariables: false,
	printPostContent: false,

        silentResponse: false,

        regexpFilterText: '$branch',
     	regexpFilterExpression: 'master'
      )

    }

    stages {
		stage ('Build Code')
		{
			steps {
				echo "Build Code ${BUILD_NUM}"
			}
		}
		stage('Deployment to Production Server')
		{
			
			steps {
				echo "Deploying to Production Server "
				
				withCredentials([azureServicePrincipal('AZURE_SER_ACCOUNT')]) {
                    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
                    sh 'az vmss nic list -g ${RESOURCE_GRP}  --vmss-name ${VMSS_NAME} --query [].{ip:ipConfigurations[0].privateIpAddress} -o tsv > ip.txt'
                    
              
                    ansiColor('xterm') {
                        ansiblePlaybook(playbook: 'azure_inv.yaml', colorized: true)
                    }
                }
                sh 'cat host.conf'
                ansiColor('xterm') {			
                    ansiblePlaybook(credentialsId: 'VMSS_KEY', inventory: 'host.conf', limit: 'vmsshosts', playbook: 'azure_deploy.yaml', colorized: true, hostKeyChecking: false)
                }
               
			}
		}
		
		
    }
	post {

        success {
		    echo "Send Success Notification"
		    slackSend color: 'good', message: "Jenkins Build Success \n\nCommit Info: ${COMMIT_INFO} \n\nBuild Url: ${BUILD_URL}"
			
		}
        failure {
			echo "Send Failure Notification"
		       slackSend color: '#FF0000', message: "Build Failure  \nCommit Info: ${COMMIT_INFO} \n\nBuild Url: ${BUILD_URL}"
		    
        }
        always {
			
            deleteDir()
            cleanWs()
           
        }
    }
}
