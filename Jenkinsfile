
pipeline {

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
    def toolbelt = tool 'toolbelt'
agent any
    stages {
    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Authorize app') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

			println rc
	}
	            stage ("info") {
            when {
		        changeRequest()
			}
			steps {
				powershell 'gci env:\\ | ft name,value -autosize'
				
                // add a ref to git config to make it aware of master
                powershell '& git config --add remote.origin.fetch +refs/heads/master:refs/remotes/origin/master'
				
                // now fetch master so you can do a diff against it 
                powershell '& git fetch --no-tags'
				
                // do the diff and set some variable based on the result
                powershell '''
					$DiffToMaster = & git diff --name-only origin/master..origin/$env:BRANCH_NAME
					Switch ($DiffToMaster) {
						'server-1607/base.json' {$env:PACK_BASE = $true}
						'server-1607/basic.json' {$env:PACK_BASIC = $true}
						'server-1607/algo.json' {$env:PACK_ALGO = $true}
						'server-1607/build.json' {$env:PACK_BUILD = $true}
						'server-1607/calc.json' {$env:PACK_CALC = $true}
					}
					gci env:/PACK_*
				'''
			}
		}
	    stage('Deploy Code'){
			// need to pull out assigned username
			if (isUnix()) {
				rmsg = sh returnStdout: true, script: "${toolbelt} force:mdapi:deploy -d force-app/main/default/. -u ${HUB_ORG}"
			}else{
			   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:mdapi:deploy -d force-app/main/default/. -u ${HUB_ORG}"
			}
			  
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
	    }
    }
        
    }
}
