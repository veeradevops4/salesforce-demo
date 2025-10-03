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
    //     ARTIFACTORY_URL = 'http://localhost:8082'

    //   // The repo in Artifactory where you want to upload
    //     ARTIFACTORY_REPO = 'salesforce-generic-local' // e.g., generic-local for generic repos
    //   // Jenkins credentials IDs for Artifactory username and password/API key
    //     // SALESFORCE_GENERIC_TOKEN = credentials('salesforce-generic-token')
    //    // Name of the Salesforce metadata ZIP file to upload
    //     REPORT_FILE = 'reports/output-report.html'  
        // Target path inside Artifactory repo (can be empty or a folder path)
        // TARGET_PATH = 'salesforce/'
        NEXUS_URL = 'http://54.91.45.21:8081/repository/sfdx/'
        NEXUS_CREDENTIALS = credentials('nexus-creds') // Jenkins credentials ID


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
                sh 'sfdx plugins:update'
                sh 'sfdx plugins:install @salesforce/sfdx-scanner'
                sh 'jfrog --version'
                  
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
                // run analysis on all apex classes
                sh'''
                sfdx scanner:run --target force-app/main/default/classes --format html --outfile reports/output-report.html
                '''

               
            }
        }
        stage('Archive reports') {
            steps {
                archiveArtifacts artifacts: 'reports/*.html', fingerprint: true
            }
        }

        // stage('Install JFrog CLI') {
        //     steps {
        //         bat '''
        //         curl -o jfrog.exe -fL https://releases.jfrog.io/artifactory/jfrog-cli/v2/2.55.1/jfrog-cli-windows-amd64/jfrog.exe

        //         '''
        //     }
        // }

        // stage('Install JFrog CLI') {
        //     steps {
        //         bat '''
        //         curl -o jfrog.exe -fL https://releases.jfrog.io/artifactory/jfrog-cli/v2-jf/jfrog.exe
        //         '''
        //     }
        // }

        // stage('Upload to Jfrog') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'artifactory-creds', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
        //             sh '''
        //                 jfrog config add art-server --url=$ARTIFACTORY_URL --user=$ART_USER --password=$ART_PASS --interactive=false --enc-password=false 
        //                 jfrog rt u "${REPORT_FILE}" "${ARTIFACTORY_REPO}/${REPORT_FILE}" --server-id=art-server
        //             '''
        //         }
        //     } 
        // }

        stage('Upload to Nexus') {
            steps {
                script {
                    def fileName = "reports/output-report.html"
                    def uploadUrl = "${NEXUS_URL}${fileName}"
                    
                    sh """
                        curl -u ${NEXUS_CREDENTIALS_USR}:${NEXUS_CREDENTIALS_PSW} \
                             --upload-file ${fileName} \
                             ${uploadUrl}
                    """
                }
            }
        }
        // stage('upload report to artifactory') {
        //     steps {
        //         script {
        //             // get a reference to the  configured artifactory server
        //             def server = Artifactory.server 'Artifactory'
        //             def buildNumber = currentBuild.number

        //             // define the upload specification
        //             def uploadSpec = """{
        //                 "files": [
        //                     { 
        //                        "pattern": "output-report.html",
        //                        "target": "cicd-generic-local/reports/${buildNumber}/"

                        
        //                     }
                        
        //                 ]
        //             }"""

        //             //upload the output-report.html file
        //             server.upload(uploadSpec)
        //         }
        //     }
        // }

        // stage('Convert and Zip') {
        //     steps {
        //         sh '''
        //             ls -la
        //             "/c/Program Files/7-Zip/7z.exe" a deploy.zip ./reports 
                    // ls -ltr
        //         '''
        //     }
        // }

        // stage('Upload to Artifactory') {
        //     steps {
        //         script {
        //             def uploadUrl = "${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/${REPORT_FILE}"
        //             echo "Uploading to: ${uploadUrl}"

        //             // Use curl to upload file to Artifactory REST API
        //              sh '''
        //                     curl -fL -H "X-JFrog-Art-Api: $SALESFORCE_GENERIC_TOKEN" \
        //                          -T reports/output-report.html \
        //                          ''' + uploadUrl + '''
        //                 '''
        //         }
        //     }
        // }
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
    


