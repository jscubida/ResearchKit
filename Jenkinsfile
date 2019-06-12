pipeline {
    agent { label 'HDS' }
    stages {
            stage('Environment Setup') {
                    steps {
                            // set up env variables for xcpretty and for xcode
                            sh 'export DEVELOPER_DIR="/Applications/XcodeYukon/Xcode.app/Contents/Developer"'
                            sh 'export LC_CTYPE=en_US.UTF-8'
                            sh 'mkdir -p output/ResearchKit/ios output/DiagnosticLogs'
                            
                            // Reset content from all the simulators
                            sh 'killall "Simulator" 2> /dev/null || true; xcrun simctl erase all'

                            // create a sentinel file to use for its modification date later
                            sh 'touch ./tests-began'

                            // print out sdk version
                            sh 'xcodebuild -sdk iphoneos -version'
                    }
            }
            stage('Build (ResearchKit iOS)') {
                    steps {
                            timeout(time: 20, unit: 'MINUTES') {
                sh 'echo "Build (ResearchKit iOS)"'
                                    sh 'set -o pipefail && xcodebuild clean build-for-testing -project ./ResearchKit.xcodeproj -scheme "ResearchKit" -destination "name=iPhone Xs" | tee output/ResearchKit/ios/build.log | /usr/local/bin/xcpretty'
                            }
                    }
            }
            stage('Test (ResearchKit iOS)') {
                    steps {
                            timeout(time: 20, unit: 'MINUTES') {
                sh 'echo "Test (ResearchKit iOS)"'
                                    sh 'set -o pipefail && xcodebuild test-without-building -project ./ResearchKit.xcodeproj -scheme "ResearchKit" -destination "name=iPhone Xs" | tee output/ResearchKit/ios/test.log | /usr/local/bin/xcpretty -r junit'
                            }
                    }
            }
            stage('Build (ORKTest iOS)') {
                    steps {
                            timeout(time: 20, unit: 'MINUTES') {
                sh 'echo "Build (ORKTest iOS)"'
                                    sh 'set -o pipefail && xcodebuild clean build-for-testing -project ./ResearchKit.xcodeproj -scheme "ORKTest" -destination "name=iPhone Xs" | tee output/ResearchKit/ios/build.log | /usr/local/bin/xcpretty'
                            }
                    }
            }
            stage('Test (ORKTest iOS)') {
                    steps {
                            timeout(time: 20, unit: 'MINUTES') {
                sh 'echo "Test (ORKTest iOS)"'
                                    sh 'set -o pipefail && xcodebuild test-without-building -project ./ResearchKit.xcodeproj -scheme "ORKTest" -destination "name=iPhone Xs" | tee output/ResearchKit/ios/test.log | /usr/local/bin/xcpretty -r junit'
                            }
                    }
            }
    }
    post {
        always {
            // copy crash logs created after the tests began
            sh 'find /Library/Logs/DiagnosticReports -type f -newer tests-began -exec cp {} output/DiagnosticLogs \\;'

            // archive all the logs
            sh 'tar -zcvf artifacts.tar.gz output'
            archiveArtifacts artifacts: 'artifacts.tar.gz', fingerprint: true
            junit 'build/reports/*.xml'
        }
    }
}

