pipeline {
    agent any

    parameters {
        string(
            name: 'RELEASE_NAME',
            defaultValue: 'demo-release',
            description: 'Name of the release'
        )
        string(
            name: 'Ahmed_Parameter',
            defaultValue: 'demo',
            description: 'Target environment'
        )
        choice(
            name: 'DEPLOY_REGION',
            choices: ['us-east-1', 'eu-central-1', 'ap-southeast-2'],
            description: 'Region to use for this test deployment'
        )
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run the test stage'
        )
        text(
            name: 'RELEASE_NOTES',
            defaultValue: 'Demo release notes\nTriggered from TeamCity',
            description: 'Multi-line release notes'
        )
    }

    stages {
        stage('Show Parameters') {
            steps {
                echo "Release name: ${params.RELEASE_NAME}"
                echo "Target environment: ${params.Ahmed_Parameter}"
                echo "Deploy region: ${params.DEPLOY_REGION}"
                echo "Run tests: ${params.RUN_TESTS}"
                echo 'Release notes:'
                echo params.RELEASE_NOTES
            }
        }

        stage('Build') {
            steps {
                echo "Building ${params.RELEASE_NAME}..."
                sh 'mkdir -p build'
                writeFile file: "build/${params.RELEASE_NAME}-v${env.BUILD_NUMBER}.txt", text: """=========================================
Release Package Artifact
=========================================
Release Name: ${params.RELEASE_NAME}
Environment : ${params.Ahmed_Parameter}
Region      : ${params.DEPLOY_REGION}
Build Number: ${env.BUILD_NUMBER}
Notes       : ${params.RELEASE_NOTES}
=========================================
"""
                sh 'echo Build completed successfully'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/**', fingerprint: true
                }
            }
        }

        stage('Tests') {
            when {
                expression {
                    return params.RUN_TESTS
                }
            }
            steps {
                echo 'Running test suites...'
                sh 'mkdir -p test-results'

                writeFile file: 'test-results/TEST-unit.xml', text: '''<?xml version="1.0" encoding="UTF-8"?>
                <testsuite name="UnitTests" tests="1" failures="0" errors="0" time="0.01">
                    <testcase name="testPasses" classname="com.demo.Unit" time="0.01"/>
                </testsuite>'''

                writeFile file: 'test-results/TEST-integration.xml', text: '''<?xml version="1.0" encoding="UTF-8"?>
                <testsuite name="IntegrationTests" tests="1" failures="1" errors="0" time="0.02">
                    <testcase name="testFails" classname="com.demo.Integration" time="0.02">
                        <failure message="Expected true but was false">Assertion error stacktrace here...</failure>
                    </testcase>
                </testsuite>'''

                writeFile file: 'test-results/TEST-ui.xml', text: '''<?xml version="1.0" encoding="UTF-8"?>
                <testsuite name="UITests" tests="1" failures="0" errors="1" time="5.00">
                    <testcase name="testTimeout" classname="com.demo.UI" time="5.00">
                        <error message="Timeout exception">Timeout stacktrace here...</error>
                    </testcase>
                </testsuite>'''
            }
            post {
                always {
                    junit 'test-results/*.xml'
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying ${params.RELEASE_NAME} to ${params.Ahmed_Parameter}..."
                sh 'echo Deployment completed successfully'
            }
        }
    }

    post {
        always {
            echo 'Jenkins build completed.'
            echo "Build number: ${env.BUILD_NUMBER}"
            cleanWs()
        }
        success {
            echo 'Pipeline finished completely successfully!'
        }
        unstable {
            echo 'Pipeline finished, but tests failed.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
