pipeline {

    agent any

    environment {
        PT_REPO = 'https://github.com/ManishaS1713/Jenkins_Pipeline_PT.git'
        JMETER_SLAVES = '192.168.1.143'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Verify JMeter') {
            steps {
                bat '''
                SET JMETER_HOME=C:\\Jmeter\\apache-jmeter-5.6.3
                C:\\Jmeter\\apache-jmeter-5.6.3\\bin\\jmeter.bat -v
                '''
            }
        }

        stage('Checkout Performance Code') {
            steps {
                git branch: 'main',
                    url: "${PT_REPO}",
                    credentialsId: 'PT_PipelineToken'
            }
        }

        stage('Run Distributed Performance Test') {
            steps {
                bat '''
                SET JMETER_HOME=C:\\Jmeter\\apache-jmeter-5.6.3

                IF EXIST performance-result.jtl del performance-result.jtl
                IF EXIST performance-report rmdir /s /q performance-report

                REM Running JMeter Distributed Test

                C:\\Jmeter\\apache-jmeter-5.6.3\\bin\\jmeter.bat -n ^
                -t prefScale.jmx ^
                -l performance-result.jtl ^
                -e -o performance-report ^
                -R 192.168.1.143
                '''
            }
        }

        stage('Generate Performance Summary') {
            steps {
                script {
                    if (fileExists('performance-result.jtl')) {

                        def raw = bat(
                            script: '''
                            powershell -Command "$data = Import-Csv 'performance-result.jtl'; 
                            $total = $data.Count; 
                            $success = ($data | Where-Object {$_.success -eq 'true'}).Count; 
                            $fail = $total - $success; 
                            $avg = [math]::Round(($data | Measure-Object -Property elapsed -Average).Average,2); 
                            Write-Output ('TOTAL=' + $total); 
                            Write-Output ('SUCCESS=' + $success); 
                            Write-Output ('FAIL=' + $fail); 
                            Write-Output ('AVG=' + $avg)"
                            ''',
                            returnStdout: true
                        ).trim()

                        def cleanLines = raw.split("\\r?\\n").findAll {
                            it.startsWith("TOTAL=") ||
                            it.startsWith("SUCCESS=") ||
                            it.startsWith("FAIL=") ||
                            it.startsWith("AVG=")
                        }

                        TOTAL = cleanLines.find { it.startsWith("TOTAL=") }?.split("=")[1]
                        SUCCESS = cleanLines.find { it.startsWith("SUCCESS=") }?.split("=")[1]
                        FAIL = cleanLines.find { it.startsWith("FAIL=") }?.split("=")[1]
                        AVG = cleanLines.find { it.startsWith("AVG=") }?.split("=")[1]

                    } else {
                        error "JMeter test failed - result file not generated"
                    }
                }
            }
        }

        stage('Publish HTML Report') {
            steps {
                publishHTML([
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
                script {
                    emailext(
                        subject: "Distributed JMeter Report",
                        body: """
<h3>Performance Test Completed 🚀</h3>

<b>Total Requests:</b> ${TOTAL} <br>
<b>Success:</b> ${SUCCESS} <br>
<b>Failures:</b> ${FAIL} <br>
<b>Avg Response Time:</b> ${AVG} ms <br>

<a href="${BUILD_URL}">View Report</a>
""",
                        to: 'manishas@ivavsys.com',
                        mimeType: 'text/html'
                    )
                }
            }
        }
    }
}
