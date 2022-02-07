#!groovy

//library identifier: 'sfdx-jenkins-libra', retriever: legacySCM([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '108f9405-04da-43c8-9fd7-7829a3adda6d', url: 'https://gitlab.com/SalesforcePatAccount/library.git']]])

node {
    
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
    def DEPLOYDIR='force-app'
    def TEST_LEVEL='RunLocalTests'
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"
    
    
    

    def toolbelt = tool 'toolbelt'
    
    
  

    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
        
    }
    
    
    stage('Calcolo diff'){
        sh '''#!/bin/sh
git fetch --tags
pwd
#Ultimo tag installato con successo
TAG=`git describe --tag --abbrev=0`
#Tag che esclude il pacchetto iniziale deloitte
#TAG=PRIMO_TAG
#Tag con tutto il branch
#TAG=START_EVO_V2.0
#FROM=beb03991237fc3ffa7c817eef1ed3da71ff680cc

FROM=`git rev-list -1 $TAG`
TO=`git rev-list -1 HEAD`

#WORKSPACE=/var/lib/jenkins/workspace/$JOB_NAME
WORKSPACE=/var/lib/jenkins/workspace/sfdx-prova_master
#WORKSPACE="."
echo git diff-tree from $FROM to $TO
echo
mkdir --parent $WORKSPACE/delta/force-app/customMetadata

git diff-tree --no-commit-id --name-only -r $TO $FROM | grep "[^.xml]$" | xargs -I {} cp --parents {} $WORKSPACE/delta/ | true
git diff-tree --no-commit-id --name-only -r $TO $FROM | grep "[.xml]$" | xargs -I {} cp --parents {} $WORKSPACE/delta/ | true

#cd $WORKSPACE/force-app/customMetadata
git diff-tree --no-commit-id --name-only -r $TO $FROM | grep customMetadata | sed "s/force-app\\/customMetadata\\///" | xargs -I {} cp --parents {} $WORKSPACE/delta/force-app/customMetadata/ | true
cd $WORKSPACE

find ./force-app -type f | grep "package.xml" | xargs -I {} cp --parents {} $WORKSPACE/delta/ | true
find ./delta/force-app -type f | sed "s/\\/delta//" | xargs -I {} cp --parents {}-meta.xml $WORKSPACE/delta/ 2>/dev/null | true
find ./delta/force-app -type f | sed "s/\\/delta//" | grep "[.xml]$" | sed "s/-meta.xml//" | xargs -I {} cp --parents {} $WORKSPACE/delta 2>/dev/null | true
if [ -d "delta/force-app/aura" ]; then
	cd ./delta/force-app/aura
	find . -type d | sed "s/.//" | xargs -I {} cp -r ../../../force-app/aura{} .
fi
#cp -R $WORKSPACE/force-app/aura $WORKSPACE/delta/force-app 2>/dev/null | true
#cp -R $WORKSPACE/force-app/email $WORKSPACE/delta/force-app 2>/dev/null | true

echo File list
find $WORKSPACE/delta/force-app -type f 
echo
echo Differences Completed
echo'''
    }

    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------

 	withEnv(["HOME=${env.WORKSPACE}"]) {	
	
	    withCredentials([file(credentialsId: 'SERVER_KEY_CREDENTIALS_ID', variable: 'SERVER_KEY_CREDENTIALS_ID')]) {
		// -------------------------------------------------------------------------
		// Authenticate to Salesforce using the server key.
		// -------------------------------------------------------------------------

		stage('Authorize to Salesforce') {
		pwd()
			rc = command "${toolbelt}/sfdx auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid 3MVG9t0sl2P.pByoE.oUJFvtbmgZssaxjcyPw5FbybL9XWAQkKNDaWG8J8Vz2iBdjCRw2ZVrWKFj4a91xOkj0 --jwtkeyfile /usr/jdoe/JWT/server.key --username patrickkitou@ten.com --setalias UAT"
		    if (rc != 0) {
			error 'Salesforce org authorization failed.'
		    }
		}


		// -------------------------------------------------------------------------
		// Deploy metadata and execute unit tests.
		// -------------------------------------------------------------------------

		stage('Deploy') {
		    rc = command "${toolbelt}/sfdx force:source:deploy --checkonly --wait 120 -p delta --targetusername patrickkitou@ten.com --testlevel RunLocalTests"
		    if (rc != 0) {
			error 'Salesforce deploy failed.'
		    }
		}


		// -------------------------------------------------------------------------
		// Example shows how to run a check-only deploy.
		// -------------------------------------------------------------------------

		//stage('Check Only Deploy') {
		//    rc = command "${toolbelt}/sfdx force:mdapi:deploy --checkonly --wait 10 --deploydir ${DEPLOYDIR} --targetusername UAT --testlevel ${TEST_LEVEL}"
		//    if (rc != 0) {
		//        error 'Salesforce deploy failed.'
		//    }
		//}
	    }
	}
}

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
		return bat(returnStatus: true, script: script);
    }
}

