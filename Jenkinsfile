import groovy.json.JsonSlurperClassic

package_dependencies = []
node ('master') {
    def PACKAGE_ID=env.PACKAGE_ID
    def SF_USERNAME=env.SF_USERNAME
    def SF_INSTANCE_URL_PROD=env.SF_INSTANCE_URL_PROD
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_DEVHUB_ORG_ALIAS='HubOrg'

    docker.image('myjenkins/node').inside {
        node ('master') {
            stage('checkout source') {
                checkout scm
            }
            
            withEnv(["HOME=${env.WORKSPACE}"]) {
                withCredentials([file(credentialsId: 'SFDX_LOGIN_SECRET', variable: 'server_key_file')]) {
                    def currentPkgSubVerId
                    stage('Authorize DevHub') {
                        echo 'Authorizing DevHub..'
                        //sh "sfdx force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL_PROD} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --setalias ${SF_DEVHUB_ORG_ALIAS}"
                        sh "sfdx force:auth:sfdxurl:store --sfdxurlfile ${server_key_file} --setalias ${SF_DEVHUB_ORG_ALIAS} --setdefaultdevhubusername --setdefaultusername"
                    }

                    stage('Identify Dependencies') {
                        echo 'Identifying Dependencies..'
                        queryOutput = sh returnStdout: true, script: "sfdx force:data:soql:query -u ${SF_DEVHUB_ORG_ALIAS} -t -q \"SELECT Id, SubscriberPackageVersionId, CreatedDate FROM Package2Version WHERE Package2Id='${PACKAGE_ID}' ORDER BY CreatedDate DESC\" --json"
                        
                        if (queryOutput) {
                            def jsonSlurper = new JsonSlurperClassic()
                            currentPkgSubVerId = jsonSlurper.parseText(queryOutput).result.records[0].SubscriberPackageVersionId
                            //echo "${currentPkgSubVerId}"
                        } else {
                            error "No package subscriber versions found for the specified package id ${PACKAGE_ID}"
                        }
                        queryOutput = sh returnStdout: true, script: "sfdx force:data:soql:query -u ${SF_DEVHUB_ORG_ALIAS} -t -q \"SELECT Dependencies FROM SubscriberPackageVersion WHERE Id='${currentPkgSubVerId}'\" --json"
                        if (queryOutput) {
                            def jsonSlurper = new JsonSlurperClassic()
                            list = jsonSlurper.parseText(queryOutput).result.records[0].Dependencies.ids
                            for (int iterator = 0; iterator < list.size(); iterator++) {
                                //echo "${list[i].subscriberPackageVersionId}"
                                package_dependencies[iterator] = list[iterator].subscriberPackageVersionId
                            }
                        } else {
                            echo "No dependencies found, move to next stage"
                        }
                    }

                    stage('Create Scratch Org') {
                        sh "sfdx force:org:create --targetdevhubusername=${SF_DEVHUB_ORG_ALIAS} --type=scratch --setalias=sorg --durationdays=1 --definitionfile=config/project-scratch-def.json --json"
                    }
                }
            }
        }
    }
}
