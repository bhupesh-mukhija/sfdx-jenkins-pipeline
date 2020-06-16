import groovy.json.JsonSlurperClassic

node ('master'){
    def PACKAGE_ID=env.SALES_PACKAGE_ID
    def SF_USERNAME=env.SF_USERNAME
    def SF_INSTANCE_URL_PROD=env.SF_INSTANCE_URL_PROD
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_DEVHUB_ORG_ALIAS='HubOrg'

    docker.image('myjenkins/node').inside {
        node ('master') {
            withEnv(["HOME=${env.WORKSPACE}"]) {
                withCredentials([file(credentialsId: 'SERVER_KEY_CREDENTALS_ID', variable: 'server_key_file')]) {
                    stage('Authorize DevHub') {
                        echo 'Authorizing DevHub..'
                        sh "sfdx force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL_PROD} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --setalias ${SF_DEVHUB_ORG_ALIAS}"
                    }

                    stage('Identify Dependencies') {
                        echo 'Identifing Dependencies..'
                        output = sh returnStdout: true, script: "sfdx force:package:version:list --packages=${PACKAGE_ID} --targetdevhubusername=${SF_DEVHUB_ORG_ALIAS} --orderby=LastModifiedDate --json"
                        def jsonSlurper = new JsonSlurperClassic()
                        def response = jsonSlurper.parseText(output);
                        echo "${response.result}"
                        echo "${response.result[0]}"
                        PACKAGE_SUBSCRIBER_VERSION_ID = response.result[0].SubscriberPackageVersionId
                        echo "${PACKAGE_SUBSCRIBER_VERSION_ID}"
                        //def querystr = "sfdx force:data:soql:query -u ${SF_DEVHUB_ORG_ALIAS} -t -q 'SELECT Dependencies FROM SubscriberPackageVersion WHERE Id=\"${PACKAGE_ID}\"' --json"
                        //echo "${querystr}"
                    }
                }
            }
        }
    }
}
