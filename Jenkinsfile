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
                    def summary = bat(
                        script: '''
                        powershell -Command "
                        $data = Import-Csv performance-result.jtl;

                        $total = $data.Count;
                        $success = ($data | Where-Object { $_.success -eq 'true' }).Count;
                        $fail = $total - $success;

                        $avg = [math]::Round(($data | Measure-Object -Property elapsed -Average).Average,2);

                        Write-Output \"TOTAL=$total\"
                        Write-Output \"SUCCESS=$success\"
                        Write-Output \"FAIL=$fail\"
                        Write-Output \"AVG=$avg\"
                        "
                        '''
                        , returnStdout: true
                    ).trim()

                    summary.split("\\r?\\n").each {
                        def parts = it.split("=")
                        env[parts[0]] = parts[1]
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
