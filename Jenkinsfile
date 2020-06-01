#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
   
   

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
	    stage('Init stage'){
            if (isUnix()) {
                println(' Convert SFDC Project to normal project')
                srccode = sh returnStdout: true, script : "sfdx force:source:convert -r force-app -d ./src"
            } else {
                println(' Convert SFDC Project to normal project')
                srccode = bat returnStdout: true, script : "sfdx force:source:convert -r force-app -d ./src"
            }
            println(srccode)
        }
        stage('Deploye Code') {
              if (isUnix()) {
                rc = sh returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} -d --instanceurl ${SFDC_HOST}"
            } else {
                rc = bat returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" -d --instanceurl ${SFDC_HOST}"
            }
           
            if (rc != 0) {
                error 'hub org authorization failed'
            }

			println rc
			
			// need to pull out assigned username
			 if(isUnix()){
                println(' Deploy the code into Scratch ORG.')
                sourcepush = sh returnStdout: true, script : "sfdx force:mdapi:deploy -d ./src -u ${HUB_ORG}"
            }else{
                println(' Deploy the code into Scratch ORG.')
                sourcepush = bat returnStdout: true, script : "sfdx force:mdapi:deploy -d ./src -u ${HUB_ORG}"
            }
			  
            printf sourcepush
            println('Hello from a Job DSL script!')
            println(sourcepush)
        }
    }
    post {

    cleanup {
        cleanWs()
    }
    always {
        bat "sfdx force:auth:logout -u ${HUB_ORG} -p" 
       
       
    }
}
}
