pipeline {
    agent any


//     withCredentials([
//         file(credentialsId: 'SFDX_JWT_KEY', variable: 'JWT_KEY_FILE'),
//         string(credentialsId: 'SFDX_CLIENT_ID', variable: 'SFDX_CLIENT_ID'),
//         string(credentialsId: 'SFDX_HUB_ORG_DH', variable: 'SFDX_HUB_ORG_DH'),
//         string(credentialsId: 'SFDC_HOST_DH', variable: 'SFDC_HOST_DH')
// ])

    environment {
        // Salesforce environment variables
        SFDX_CLIENT_ID     = credentials('SFDX_CLIENT_ID')
        SFDX_HUB_ORG_DH    = credentials('SFDX_HUB_ORG_DH') // can be alias or username
        SFDX_JWT_KEY       = credentials('SFDX_JWT_KEY') // private key for JWT auth
        SFDC_HOST_DH       = credentials('SFDC_HOST_DH')
        // Your JFrog Artifactory base URL
        ARTIFACTORY_URL = 'http://localhost:8082/artifactory'

      // The repo in Artifactory where you want to upload
        ARTIFACTORY_REPO = 'salesforce-generic-local' // e.g., generic-local for generic repos
      // Jenkins credentials IDs for Artifactory username and password/API key
        SALESFORCE-GENERIC-TOKEN = credentials('salesforce-generic-token')
       // Name of the Salesforce metadata ZIP file to upload
        SF_ZIP = 'sfdx-demo.zip'
        // Target path inside Artifactory repo (can be empty or a folder path)
        TARGET_PATH = 'salesforce/'


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
                sh 'sfdx --version'
                sh 'sfdx plugins:install @salesforce/sfdx-scanner'

                  
            }
        }
        

        stage('Authenticate with Salesforce org') {
            steps {
                withCredentials([file(credentialsId: 'SFDX_JWT_KEY', variable: 'JWT_KEY_FILE')]) {
                    sh '''
                        echo "Authenticating to Salesforce..."
                        sfdx auth:jwt:grant \
                            --client-id $SFDX_CLIENT_ID \
                            --jwt-key-file $SFDX_JWT_KEY \
                            --username $SFDX_HUB_ORG_DH \
                            --instance-url $SFDC_HOST_DH
                    '''
                }
            }
        }

        stage('Static code analysis-PMD') {
            steps {
                bat 'echo Current directory: && cd'
                bat 'dir'
                bat '''
                    C:\\pmd-bin-6.55.0\\bin\\pmd.bat ^
                        -language apex ^
                        -d force-app\\main\\default\\classes ^
                        -R apex-ruleset.xml ^
                        -f text ^
                        -r pmd-report.txt
                '''
               
            }
        }

        stage('package metadata') {
            steps {
                sh '''
                    ls -la
                    "/c/Program Files/7-Zip/7z.exe" a sfdx-demo.zip ./force-app ./manifest ./sfdx-project.json
                '''
            }
        }

        stage('Upload to Artifactory') {
            steps {
                script {
                    def uploadUrl = "${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/${TARGET_PATH}${SF_ZIP}"
                    echo "Uploading to: ${uploadUrl}"

                    // Use curl to upload file to Artifactory REST API
                    sh """
                        curl -u ${SALESFORCE-GENERIC-TOKEN}  -T ${SF_ZIP} "${uploadUrl}"
                    """
                }
            }
        }
    }
}
    



        // stage('Authenticate with Salesforce org') {
        //     steps {
        //          sh '''
        //             echo "Authenticating to Salesforce..."
        //             sfdx auth:jwt:grant \
        //                 --client-id $SFDX_CLIENT_ID \
        //                 --jwt-key-file $JWT_KEY_FILE \
        //                 --username $SFDX_HUB_ORG_DH \
        //                 --instance-url $SFDC_HOST_DH
        //         '''
        //     }
        // }

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
    


