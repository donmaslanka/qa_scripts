pipeline {
    agent { label 'ec2-agent-01' }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        KATALON_API_KEY = credentials('katalon-api-key')
        KATALON_PROJECT  = 'RESPONSE.prj'
        KATALON_SUITE    = 'Test Suites/Headless-QA'
        KATALON_SUITE_TYPE = 'collection'
        KATALON_BROWSER  = 'Chrome (headless)'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Preflight') {
            steps {
                sh '''
                    set -eux
                    echo "Workspace: $WORKSPACE"
                    hostname
                    pwd
                    ls -la
                    which git || true
                    git --version || true
                    which java || true
                    java -version || true
                    which katalonc || true

                    test -f "$WORKSPACE/$KATALON_PROJECT"
                '''
            }
        }

        stage('Run Katalon') {
            steps {
                sh '''
                    set -eux

                    # Clean any leftover Katalon workspace artifacts from prior runs
                    rm -rf "$WORKSPACE/workspace" || true

                    katalonc \
                      -noSplash \
                      -runMode=console \
                      -projectPath="$WORKSPACE/$KATALON_PROJECT" \
                      -testSuiteCollectionPath="$KATALON_SUITE" \
                      -apiKey="$KATALON_API_KEY" \
                      -orgID="2333388" \
                      -retry=0 \
                      --config -webui.autoUpdateDrivers=true \
                      "-chromiumOptions=--disable-dev-shm-usage,--no-sandbox,--disable-gpu,--disable-extensions"
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'Reports/**/*,report/**/*,reports/**/*', allowEmptyArchive: true
            junit testResults: '**/*.xml', allowEmptyResults: true
            // Clean up disk-heavy generated folders after archiving
            sh '''
                echo "=== Post-build cleanup ==="
                rm -rf "$WORKSPACE/Reports" || true
                rm -rf "$WORKSPACE/workspace" || true
                rm -rf "$WORKSPACE/Libs/Temp*" || true
                rm -rf /tmp/session-* || true
                echo "Cleanup complete. Disk usage:"
                df -h /
            '''
        }
        success {
            echo 'Katalon execution completed successfully.'
        }
        failure {
            echo 'Katalon execution failed. Check console output and archived reports.'
        }
    }
}