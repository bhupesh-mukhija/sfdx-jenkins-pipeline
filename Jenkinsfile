import groovy.json.JsonSlurperClassic

package_dependencies = []

import groovy.json.JsonSlurperClassic

package_dependencies = []
node ('master') {
    def PACKAGE_ID=env.PACKAGE_ID
    def SF_USERNAME=env.SF_USERNAME
    def SF_INSTANCE_URL_PROD=env.SF_INSTANCE_URL_PROD
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_DEVHUB_ORG_ALIAS='HubOrg'
    
    stage ('Parallel Builds')
    parallel(
        "Build ScratchOrg" : {
            node ('master'){
                stage('checkout source ScratchOrg Build') {
                    checkout scm
                }
                echo 'Running Build ScratchOrg'
                docker.image('myjenkins/node').inside {
                    withEnv(["HOME=${env.WORKSPACE}"]) {
                        withCredentials([file(credentialsId: 'SFDX_LOGIN_SECRET1', variable: 'server_key_file')]) {
                            def currentPkgSubVerId
                            stage('Authorize DevHub') {
                                echo 'Authorizing DevHub..'
                                sh "sfdx force:auth:sfdxurl:store --sfdxurlfile ${server_key_file} --setalias ${SF_DEVHUB_ORG_ALIAS} --setdefaultdevhubusername --setdefaultusername"
                            }
                            stage('Create Scratch Org') {
                                sh "sfdx force:org:create --targetdevhubusername=${SF_DEVHUB_ORG_ALIAS} --type=scratch --setalias=${SCRATCH_ORG_ALIAS} --durationdays=1 --definitionfile=config/project-scratch-def.json --json"
                            }
                        }
                    }
                }  
            }
        },
        "Build Sandbox" : {
            node ('master'){
                stage('checkout source Sandbox Build') {
                    checkout scm
                }
                echo 'Running Build Sandbox'
                docker.image('myjenkins/node').inside {
                    withEnv(["HOME=${env.WORKSPACE}"]) {
                        withCredentials([file(credentialsId: 'SFDX_LOGIN_SECRET1', variable: 'server_key_file')]) {
                            def currentPkgSubVerId
                            stage('Authorize DevHub') {
                                echo 'Authorizing DevHub..'
                                sh "sfdx force:auth:sfdxurl:store --sfdxurlfile ${server_key_file} --setalias ${SF_DEVHUB_ORG_ALIAS} --setdefaultdevhubusername --setdefaultusername"
                            }
                            stage('Authorize Sandbox') {
                                sh "sfdx force:org:list --all"
                            }
                        }
                    }
                }
            }
        }
    )
}