pipeline {

    agent any   // Jenkins Master will trigger pipeline

    environment {
        PT_REPO = 'https://github.com/ManishaS1713/Jenkins_Pipeline_PT.git'

        // ✅ Define Slave IPs (JMeter servers)
        JMETER_SLAVES = '192.168.1.5'
        
        // ✅ JMeter path (same path must exist on master & slaves)
        JMETER_HOME = 'C:\\Jmeter\\apache-jmeter-5.6.3\\bin'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Verify JMeter on Master') {
            steps {
                // ✅ Verify JMeter installed on Master
                bat "\"${JMETER_HOME}\\jmeter.bat\" -v"
            }
        }

        stage('Checkout Performance Code') {
            steps {
                git branch: 'main',
                    url: "${PT_REPO}",
                    credentialsId: 'PT_PipelineToken'
            }
        }

        stage('Start JMeter Servers on Slaves') {
            steps {
                script {
                    // ✅ This assumes JMeter server already running on slaves
                    // OR you can start manually using jmeter-server.bat on slave machines
                    echo "Ensure JMeter Server is running on slaves: ${JMETER_SLAVES}"
                }
            }
        }

        stage('Run Distributed Performance Test') {
            steps {
                bat """
                IF EXIST performance-result.jtl del performance-result.jtl
                IF EXIST performance-report rmdir /s /q performance-report

                // ✅ Run JMeter in Distributed Mode
                // -R = Remote servers (slaves)
                "${JMETER_HOME}\\jmeter.bat" -n ^
                -t prefScale.jmx ^
                -l performance-result.jtl ^
                -e -o performance-report ^
                -R ${JMETER_SLAVES}
                """
            }
        }

        stage('Generate Performance Summary') {
            steps {
                script {
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

                    // ✅ Extract values for reporting
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
                        subject: "Distributed JMeter Performance Report",
                        body: """
<h3>Distributed Performance Test Completed 🚀</h3>

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
