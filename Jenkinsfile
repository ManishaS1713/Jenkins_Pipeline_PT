pipeline {

    agent any

    environment {
        PT_REPO = 'https://github.com/ManishaS1713/Jenkins_Pipeline_PT.git'
        JMETER_SLAVES = '192.168.0.147'
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
                -R 192.168.0.147
                '''
            }
        }

       stage('Generate Performance Summary') {
    steps {
        script {
            if (fileExists('performance-result.jtl')) {
                bat '''
                powershell -Command "
                $data = Import-Csv 'performance-result.jtl';
                $total = $data.Count;
                $success = ($data | Where-Object {$_.success -eq 'true'}).Count;
                Write-Output ('Total Requests: ' + $total);
                Write-Output ('Successful Requests: ' + $success)
                "
                '''
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
