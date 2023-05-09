pipeline {
    agent {
        label 'agent'
    }
    environment {
        APP_NAME = <sample app>
        SCHEME_NAME = <scheme>
        CONFIGURATION = "Release"
        SIGNING_IDENTITY = <Distribution_cert>
        PROVISIONING_PROFILE = <Provisional profile>
        EXPORT_OPTIONS_PLIST = "${WORKSPACE}/exportOptions.plist"
        EXPORT_FOLDER_NAME = "Archive"
        ARCHIVE_PATH = "${WORKSPACE}/${EXPORT_FOLDER_NAME}/${APP_NAME}.xcarchive"
        EXPORT_PATH = "${WORKSPACE}/${EXPORT_FOLDER_NAME}/${APP_NAME}"
        KEYCHAIN_ACCESS_TOKEN = <KEYCHAIN_ACCESS_TOKEN>
    }
    stages {
        stage('Create Archive folder') {
            steps {
                sh 'if [ -d "Archive" ]; then rm -rf Archive; fi'
                sh 'mkdir Archive'
            }
        }
        stage('Archive') {
            steps {
                sh "mkdir -p '$EXPORT_FOLDER_NAME'"
                // Give jenkins permission to access keychain
                sh "security unlock-keychain -p '${KEYCHAIN_ACCESS_TOKEN}' ~/Library/Keychains/login.keychain"
                // archive
                sh "xcodebuild -archivePath '${ARCHIVE_PATH}' \
                               -scheme '${SCHEME_NAME}' \
                               -configuration '${CONFIGURATION}' \
                               clean archive \
                               CODE_SIGN_IDENTITY='${SIGNING_IDENTITY}' \
                               PROVISIONING_PROFILE='${PROVISIONING_PROFILE}' \
                               CODE_SIGN_STYLE=Manual \
                               -allowProvisioningUpdates"
            }
        }

        stage('Export') {
            steps {
                sh """xcodebuild -exportArchive -archivePath '${ARCHIVE_PATH}' \
                               -exportPath '${EXPORT_PATH}' \
                               -exportOptionsPlist '${EXPORT_OPTIONS_PLIST}' \
                               -allowProvisioningUpdates"""

                sh '''#!/bin/bash
                DATE_OF_COMMIT=$(date -r $(git log -1 --format="%at"))
                /opt/homebrew/bin/jq -n \
                --arg gitcommit $GIT_COMMIT \
                --arg gitbranch $GIT_BRANCH \
                --arg giturl $GIT_URL \
                --arg dateofcommit "$DATE_OF_COMMIT" \
                --arg buildid $BUILD_ID \
                --arg buildurl $BUILD_URL \
                \'{"Git Commit":$ARGS.named["gitcommit"],"Git Branch":$ARGS.named["gitbranch"],"Git-URL":$ARGS.named["giturl"],"Date of Commit":$ARGS.named["dateofcommit"],"Build-ID":$ARGS.named["buildid"],"Build-URL":$ARGS.named["buildurl"]}\' > buildinfo.json
                '''
            }
        }
    }

    post
    {
        always 
        {
            // Archive ipa file and buildjson
            archiveArtifacts artifacts: "sample.ipa, buildinfo.json", fingerprint: false
        }
    }
}
