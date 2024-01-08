node {
    try {
        // Declare the Docker image for the entire pipeline
        docker.image("python:3.12.1").inside("--user 0:0") {

            // Build stage
            stage('Build') {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }

            // Test stage
            stage('Test') {
                docker.image('qnib/pytest').inside {
                    sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
                }
                junit 'test-reports/results.xml'
            }

            // Deploy stage
            stage('Deploy') {
                sh 'pip install pyinstaller --user'
                sh 'pyinstaller --onefile sources/add2vals.py'
                archiveArtifacts 'dist/add2vals'
            }
        }

        // Additional steps outside Docker
        steps {
            sh 'git clean -fdx'
        }

        // Post-build cleanup
        post {
            cleanup {
                cleanWs()
            }
        }

    } catch (Exception e) {
        echo "Pipeline failed: ${e.message}"
        currentBuild.result = 'FAILURE'
    }
}
