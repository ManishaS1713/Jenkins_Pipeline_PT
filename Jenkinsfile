pipeline {

    agent any

    environment {
        PT_REPO = 'https://github.com/ManishaS1713/Jenkins_Pipeline_PT.git'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Verify JMeter') {
            steps {
                bat '"C:\\Jmeter\\apache-jmeter-5.6.3\\bin\\jmeter.bat" -v'
            }
        }

        stage('Checkout Performance Code') {
            steps {
                git branch: 'main',
                    url: "${PT_REPO}",
                    credentialsId: 'PT_PipelineToken'
            }
        }

        stage('Run Performance Tests') {
            steps {
                bat '''
                IF EXIST performance-result.jtl del performance-result.jtl
                IF EXIST performance-report rmdir /s /q performance-report

                C:\\Jmeter\\apache-jmeter-5.6.3\\bin\\jmeter.bat -n ^
                -t prefScale.jmx ^
                -l performance-result.jtl ^
                -e -o performance-report
                '''
            }
        }

        stage('Generate Performance Summary') {
    steps {
        script {
            def raw = bat(
                script: '''
                powershell -Command "$data = Import-Csv 'performance-result.jtl'; $total = $data.Count; $success = ($data | Where-Object {$_.success -eq 'true'}).Count; $fail = $total - $success; $avg = [math]::Round(($data | Measure-Object -Property elapsed -Average).Average,2); Write-Output ('TOTAL=' + $total); Write-Output ('SUCCESS=' + $success); Write-Output ('FAIL=' + $fail); Write-Output ('AVG=' + $avg)"
                ''',
                returnStdout: true
            ).trim()

            echo "RAW OUTPUT:\n${raw}"

            // ✅ Filter only valid lines
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
                        subject: "JMeter Performance Test Report",
                        body: """
<h3>Performance Test Completed ✅</h3>

<b>Total Requests:</b> ${TOTAL} <br>
<b>Success:</b> ${SUCCESS} <br>
<b>Failures:</b> ${FAIL} <br>
<b>Avg Response Time:</b> ${AVG} ms <br>

<br><b>👉 View Full Report:</b><br>
<a href="${BUILD_URL}">${BUILD_URL}</a>
""",
                        to: 'manishas@ivavsys.com',
                        mimeType: 'text/html'
                    )
                }
            }
        }

    }
}
