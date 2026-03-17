pipeline {

    agent any

    environment {
        PERF_EMAIL = 'manishas@ivavsys.com'
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

        stage('Zip HTML Report') {
            steps {
                bat '''
                powershell Compress-Archive -Path performance-report -DestinationPath performance-report.zip -Force
                '''
            }
        }

        stage('Archive Performance Results') {
            steps {
                archiveArtifacts artifacts: 'performance-result.jtl, performance-report.zip', fingerprint: true
            }
        }
        
        stage('Check Files') {
            steps {
                bat 'dir'
            }
        }
        
        stage('Email After Performance') {
            steps {
                emailext(
                    subject: "JMeter Performance Test Report",
                    body: """
Performance Test Execution Completed.

Jenkins Build Report:
${BUILD_URL}

HTML report attached in ZIP format.
""",
                    to: env.PERF_EMAIL,
                    attachmentsPattern: '**/performance-report.zip',
                    mimeType: 'text/html',
                    attachLog: true
                )
            }
        }

    }
}
