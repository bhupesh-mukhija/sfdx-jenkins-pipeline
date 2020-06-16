import groovy.json.JsonSlurperClassic

package_dependencies = []
node ('master') {
    def PACKAGE_ID=env.PACKAGE_ID
    def SF_USERNAME=env.SF_USERNAME
    def SF_INSTANCE_URL_PROD=env.SF_INSTANCE_URL_PROD
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_DEVHUB_ORG_ALIAS='HubOrg'
    def SCRATCH_ORG_ALIAS='sorg'
    // TODO: BELOW SHOULD BE PARAMETERISED
    def INSTALL_WAIT=10
    // TODO: INSTALLATION KEYS

    docker.image('myjenkins/node').inside {
        node ('master') {
            stage('checkout source') {
                checkout scm
            }
            
            withEnv(["HOME=${env.WORKSPACE}"]) {
                withCredentials([file(credentialsId: 'SFDX_LOGIN_SECRET1', variable: 'server_key_file')]) {
                    def currentPkgSubVerId
                    stage('Authorize DevHub') {
                        echo 'Authorizing DevHub..'
                        //sh "sfdx force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL_PROD} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --setalias ${SF_DEVHUB_ORG_ALIAS}"
                        sh "sfdx force:auth:sfdxurl:store --sfdxurlfile ${server_key_file} --setalias ${SF_DEVHUB_ORG_ALIAS} --setdefaultdevhubusername --setdefaultusername"
                    }

                    stage('Identify Dependencies') {
                        echo 'Identifying Dependencies..'
                        echo 'Find latest package subscriber versions..'
                        queryOutput = sh returnStdout: true, script: "sfdx force:data:soql:query -u ${SF_DEVHUB_ORG_ALIAS} -t -q \"SELECT Id, SubscriberPackageVersionId, CreatedDate FROM Package2Version WHERE Package2Id='${PACKAGE_ID}' ORDER BY CreatedDate DESC\" --json"
                        
                        if (queryOutput) {
                            def jsonSlurper = new JsonSlurperClassic()
                            def jsonOutput = jsonSlurper.parseText(queryOutput)
                            if (jsonOutput.status == 0 && jsonOutput.result.size > 0) {
                                currentPkgSubVerId = jsonOutput.result.records[0].SubscriberPackageVersionId
                            } else {
                                echo "No package subscriber versions found for the specified package id ${PACKAGE_ID}"
                            }
                            //echo "${currentPkgSubVerId}"
                        } else {
                            echo "No package subscriber versions found for the specified package id ${PACKAGE_ID}"
                        }
                        if (currentPkgSubVerId) {
                            queryOutput = sh returnStdout: true, script: "sfdx force:data:soql:query -u ${SF_DEVHUB_ORG_ALIAS} -t -q \"SELECT Dependencies FROM SubscriberPackageVersion WHERE Id='${currentPkgSubVerId}'\" --json"
                            if (queryOutput) {
                                def jsonSlurper = new JsonSlurperClassic()
                                def jsonOutput = jsonSlurper.parseText(queryOutput)
                                if (jsonOutput.status == 0 && jsonOutput.result.size > 0) {
                                    def list = jsonOutput.result.records[0].Dependencies.ids
                                    for (int iterator = 0; iterator < list.size(); iterator++) {
                                        package_dependencies[iterator] = list[iterator].subscriberPackageVersionId
                                    }
                                    // TODO: FETCH INSTALLATION KEYS WHEREVER APPLICABLE
                                } else {
                                    echo "No dependencies found, move to next stage"
                                }
                            } else {
                                echo "No dependencies found, move to next stage"
                            }
                        }
                    }

                    stage('Create Scratch Org and install dependencies') {
                        sh "sfdx force:org:create --targetdevhubusername=${SF_DEVHUB_ORG_ALIAS} --type=scratch --setalias=${SCRATCH_ORG_ALIAS} --durationdays=1 --definitionfile=config/project-scratch-def.json --json"

                        for (int iterator = 0; iterator < package_dependencies.size(); iterator++) {
                            // TODO: DECIDE ON --upgradetype, --securitytype and way to parameterize these values
                            sh "sfdx force:package:install --targetusername=${SCRATCH_ORG_ALIAS} --package=${package_dependencies[iterator]} --noprompt --upgradetype=DeprecateOnly --securitytype=AllUsers --apexcompile=all"
                            //package_dependencies[iterator]
                        }
                    }

                    stage('Validate Source') {
                        sh "sfdx force:source:push --targetusername=${SF_DEVHUB_ORG_ALIAS}"
                    }
                }
            }
        }
    }
}
