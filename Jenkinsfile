pipeline {
    agent any

    environment {
        // Salesforce environment variables
        SFDX_CLIENT_ID     = credentials('CONNECTED_APP_CONSUMER_KEY_DH')
        SFDX_HUB_ORG_DH    = credentials('HUB_ORG_DH') // can be alias or username
        SFDX_JWT_KEY       = credentials('JWT_CRED_ID_DH') // private key for JWT auth
        SFDC_HOST_DH       = credentials('SFDC_HOST_DH')
        // SFDX_ORG_ALIAS     = 'myOrg' // You can use any alias
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Install Salesforce CLI') {
            steps {
                // sh 'npm install sfdx-cli --global'
                sh 'sf --version'

                  
            }
        }

        stage('Authenticate with Salesforce org') {
            steps {
                sh """
                    echo "$SFDX_JWT_KEY" > server.key
                    sfdx auth:jwt:grant \
                        --clientid $SFDX_CLIENT_ID \
                        --jwtkeyfile $SFDX_JWT_KEY \
                        --username $SFDX_HUB_ORG_DH \
                        --instanceurl $SFDC_HOST_DH
                """
            }
        }

        // stage('Deploy to Salesforce Org') {
        //     steps {
        //         sh "sfdx force:source:deploy -p force-app/main/default -u $SFDX_ORG_ALIAS --checkonly --verbose"
        //         sh "sfdx force:source:deploy -p force-app/main/default -u $SFDX_ORG_ALIAS --wait 10"
        //     }
        // }

        // stage('Run Apex Tests') {
        //     steps {
        //         sh "sfdx force:apex:test:run --resultformat human --wait 10 --codecoverage --u $SFDX_ORG_ALIAS"
        //     }
        // }
    }
}

