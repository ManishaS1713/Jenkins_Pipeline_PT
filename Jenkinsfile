pipeline {
    agent any   // Run pipeline on any available Jenkins agent

    environment {
        PT_REPO = 'https://github.com/ManishaS1713/Jenkins_Pipeline_PT.git'  // GitHub repo URL
        JMETER_HOME = 'C:\\Jmeter\\apache-jmeter-5.6.3'  // JMeter installation path
        SLAVE_IP = '192.168.0.147'  // Remote JMeter slave IP for distributed testing
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()  // Clean previous build files to avoid conflicts
            }
        }

        stage('Checkout Code') {
            steps {
                // Pull latest code (JMX + Jenkinsfile) from GitHub
                git branch: 'main',
                    url: "${PT_REPO}",
                    credentialsId: 'PT_PipelineToken'
            }
        }

        stage('Verify JMeter') {
            steps {
                bat '''
                // Set JMeter path and verify installation
                SET JMETER_HOME=%JMETER_HOME%
                %JMETER_HOME%\\bin\\jmeter.bat -v
                '''
            }
        }

        stage('Run Distributed Performance Test') {
            steps {
                bat '''
                // Set JMeter path
                SET JMETER_HOME=%JMETER_HOME%

                // Delete old result and report folders if exist
                IF EXIST performance-result.jtl del performance-result.jtl
                IF EXIST performance-report rmdir /s /q performance-report

                // Run JMeter in non-GUI mode with distributed setup
                %JMETER_HOME%\\bin\\jmeter.bat -n ^
                -t prefScale.jmx ^                     // Test plan
                -l performance-result.jtl ^            // Result file
                -e -o performance-report ^             // HTML report output
                -R %SLAVE_IP% ^                       // Remote slave execution
                -Jjmeter.save.saveservice.output_format=csv  // Save results in CSV
                '''
            }
        }

        stage('Generate Performance Summary') {
            steps {
                script {
                    // Check if result file exists before processing
                    if (fileExists('performance-result.jtl')) {

                        // Execute PowerShell to calculate total & successful requests
                        def output = bat(
                            script: '''
                            @echo off
                            powershell -Command "$data = Import-Csv 'performance-result.jtl'; $total = $data.Count; $success = ($data | Where-Object {$_.success -eq 'true'}).Count; Write-Output \"$total,$success\""
                            ''',
                            returnStdout: true
                        ).trim()

                        // Extract only the actual result (last line)
                        def cleanOutput = output.tokenize('\\n')[-1].trim()

                        // Split values into TOTAL and SUCCESS
                        def parts = cleanOutput.split(',')
                        env.TOTAL = parts[0]
                        env.SUCCESS = parts[1]

                        // Print summary in Jenkins console
                        echo "Total Requests: ${env.TOTAL}"
                        echo "Successful Requests: ${env.SUCCESS}"
                    }
                }
            }
        }

        stage('Publish HTML Report') {
            steps {
                // Publish JMeter HTML report in Jenkins UI
                publishHTML(target: [
                    reportDir: 'performance-report',
                    reportFiles: 'index.html',
                    reportName: 'JMeter Performance Report',
                    keepAll: true,
                    alwaysLinkToLastBuild: true,
                    allowMissing: false
                ])
            }
        }

        stage('Email After Performance') {
            steps {
                // Send email with test summary
                emailext(
                    subject: "JMeter Test Result - Build #${BUILD_NUMBER}",
                    body: """
                    <h3>Performance Test Summary</h3>
                    <p><b>Total Requests:</b> ${env.TOTAL}</p>
                    <p><b>Successful Requests:</b> ${env.SUCCESS}</p>
                    <p>Check detailed report in Jenkins.</p>
                    """,
                    to: "manishas@ivavsys.com",
                    mimeType: 'text/html'
                )
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'  // Always runs after pipeline
        }
        success {
            echo 'Performance test passed successfully!'  // On success
        }
        failure {
            echo 'Performance test failed!'  // On failure
        }
    }
}
