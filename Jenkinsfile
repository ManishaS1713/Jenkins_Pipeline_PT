pipeline {

    agent any

    environment {
        PERF_EMAIL = 'manishas@ivavsys.com'
        PT_REPO = 'https://github.com/ManishaS1713/Jenkins_Pipeline_PT.git'
    }
    
stage('Clean Workspace') {
    steps {
        cleanWs()
    }
}
    stages {

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

        stage('Email After Performance') {
            steps {
                emailext(
                    subject: "JMeter Performance Test Report",
                    body: """
Performance Test Completed.

View Report:
${BUILD_URL}
""",
                    to: "${PERF_EMAIL}",
                    attachmentsPattern: 'performance-report/index.html'
                )
            }
        }

    }
}
