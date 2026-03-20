pipeline {
    agent any

    environment {
        JMETER_HOME = 'C:\\Jmeter\\apache-jmeter-5.6.3'
        SLAVE_IP = '192.168.0.147'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git credentialsId: 'PT_PipelineToken',
                    url: 'https://github.com/ManishaS1713/Jenkins_Pipeline_PT.git'
            }
        }

        stage('Verify JMeter') {
            steps {
                bat '''
                SET JMETER_HOME=%JMETER_HOME%
                %JMETER_HOME%\\bin\\jmeter.bat -v
                '''
            }
        }

        stage('Run Distributed Performance Test') {
            steps {
                bat '''
                SET JMETER_HOME=%JMETER_HOME%

                IF EXIST performance-result.jtl del performance-result.jtl
                IF EXIST performance-report rmdir /s /q performance-report

                %JMETER_HOME%\\bin\\jmeter.bat -n ^
                -t prefScale.jmx ^
                -l performance-result.jtl ^
                -e -o performance-report ^
                -R %SLAVE_IP% ^
                -Jjmeter.save.saveservice.output_format=csv
                '''
            }
        }

        stage('Generate Performance Summary') {
            steps {
                script {
                    if (fileExists('performance-result.jtl')) {

                        def output = bat(
                            script: '''
                            powershell -Command "$data = Import-Csv 'performance-result.jtl'; $total = $data.Count; $success = ($data | Where-Object {$_.success -eq 'true'}).Count; Write-Output \"$total,$success\""
                            ''',
                            returnStdout: true
                        ).trim()

                        def parts = output.split(',')
                        env.TOTAL = parts[0]
                        env.SUCCESS = parts[1]

                        echo "Total Requests: ${env.TOTAL}"
                        echo "Successful Requests: ${env.SUCCESS}"
                    }
                }
            }
        }

        stage('Publish HTML Report') {
            steps {
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
                emailext(
                    subject: "JMeter Test Result - Build #${BUILD_NUMBER}",
                    body: """
                    <h3>Performance Test Summary</h3>
                    <p><b>Total Requests:</b> ${env.TOTAL}</p>
                    <p><b>Successful Requests:</b> ${env.SUCCESS}</p>
                    <p>Check detailed report in Jenkins.</p>
                    """,
                    to: "your-email@example.com"
                )
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Performance test passed successfully!'
        }
        failure {
            echo 'Performance test failed!'
        }
    }
}
