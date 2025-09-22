pipeline {
    agent any

    environment {
        // Salesforce environment variables
        SFDX_CLIENT_ID     = credentials('salesforce-client-id')
        SFDX_HUB_USERNAME  = credentials('salesforce-username') // can be alias or username
        SFDX_JWT_KEY       = credentials('salesforce-jwt-key') // private key for JWT auth
        SFDX_ORG_ALIAS     = 'myOrg' // You can use any alias
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Install Salesforce CLI') {
            steps {
                sh 'npm install sfdx-cli --global'
            }
        }

        stage('Authenticate with Salesforce') {
            steps {
                sh """
                    echo "$SFDX_JWT_KEY" > server.key
                    sfdx auth:jwt:grant \
                        --clientid $SFDX_CLIENT_ID \
                        --jwtkeyfile server.key \
                        --username $SFDX_HUB_USERNAME \
                        --setalias $SFDX_ORG_ALIAS \
                        --instanceurl https://login.salesforce.com
                """
            }
        }

        stage('Deploy to Salesforce Org') {
            steps {
                sh "sfdx force:source:deploy -p force-app/main/default -u $SFDX_ORG_ALIAS --checkonly --verbose"
                sh "sfdx force:source:deploy -p force-app/main/default -u $SFDX_ORG_ALIAS --wait 10"
            }
        }

        stage('Run Apex Tests') {
            steps {
                sh "sfdx force:apex:test:run --resultformat human --wait 10 --codecoverage --u $SFDX_ORG_ALIAS"
            }
        }
    }

