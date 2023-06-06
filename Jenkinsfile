// Jenkinsfile solution based on the pipeline plugin:
// https://www.jenkins.io/solutions/pipeline/

// Results are output in SARIF format & processed using the Warnings NG plugin:
// https://plugins.jenkins.io/warnings-ng/

// Reports are produced in HTML format & processed using the HTML Publisher plugin:
// https://plugins.jenkins.io/htmlpublisher/

// TODO: follow step-by-step guide on setting this up in Jenkins:
//

pipeline {
    agent any

    tools {
        nodejs 'NodeJS v19.2.0'
    }
    stages {
        stage('Git Clone') {
            steps {
                git url: 'https://github.com/billytronics/goof.git', branch: 'main'
            }
        }

        // Install the Snyk CLI with npm. For more information, check:
        // https://docs.snyk.io/snyk-cli/install-the-snyk-cli
        // Install the Snyk JSON to HTML Mapper. For more information, check:
        // https://github.com/snyk/snyk-to-html
        stage('Install snyk CLI') {
            steps {
                script {
                    sh 'npm install -g snyk'
                    sh 'npm install -g snyk-to-html'
                }
            }
        }

        // Authorize the Snyk CLI
        // Persist Snyk Token as SNYK_TOKEN in Jenkins Credentials Store as Secret text
        stage('Authorize Snyk CLI') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh 'snyk auth ${SNYK_TOKEN}'
                }
            }
        }

        stage('Build App') {
            steps {
                // Replace this with your build instructions, as necessary.
                sh 'echo no-op'
            }
        }

        // Test with Snyk
        stage('Security Testing') {
            parallel {
                stage('SAST: Snyk Code Test') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            sh 'snyk code test --sarif-file-output=reports/results-code.sarif --json-file-output=reports/results-code.json'
                        }
                        // Uploads sarif data to Jenkins
                        recordIssues  tool: sarif(name: 'Snyk Code', id: 'snyk-code', pattern: 'reports/results-code.sarif')
                        sh 'snyk-to-html -i reports/results-code.json -o reports/results-code.html'
                        // Uploads html report to Jenkins
                        publishHTML (target : [allowMissing: true,
                                               keepAll: true,
                                               reportDir: 'reports',
                                               reportFiles: 'results-code.html',
                                               reportName: 'Snyk Code Report',
                                               reportTitles: 'Snyk Code Report: SAST Results'])
                    }
                }
                stage('SCA: Snyk Open Source Test') {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            sh 'snyk test --sarif-file-output=reports/results-oss.sarif --json-file-output=reports/results-oss.json'
                        }
                        // Uploads sarif data to Jenkins
                        recordIssues  tool: sarif(name: 'Snyk Open Source', id: 'snyk-oss', pattern: 'reports/results-oss.sarif')
                        sh 'snyk-to-html -i reports/results-oss.json -o reports/results-oss.html'
                        // Uploads html report to Jenkins
                        publishHTML (target : [allowMissing: true,
                                               keepAll: true,
                                               reportDir: 'reports',
                                               reportFiles: 'results-oss.html',
                                               reportName: 'Snyk Open Source Report',
                                               reportTitles: 'Snyk Open Source Report: SCA Results'])
                    }
                }
            }
        }
    }
}