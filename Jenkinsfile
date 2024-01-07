node {
    try {
        stage('Build') {
            docker.image('python:3.12.1-alpine3.19').inside {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        stage('Test') {
            docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            junit 'test-reports/results.xml'
        }

        stage('Deploy') {
            docker.image('python:3.12.1-alpine3.19').inside {
                // Set a writable directory as HOME
                sh 'export HOME=/tmp'
                
                // Install pyinstaller with user-level permissions
                sh 'pip install --user pyinstaller'

                // Run pyinstaller using the user-level bin directory
                sh '$HOME/.local/bin/pyinstaller --onefile sources/add2vals.py'

                archiveArtifacts 'dist/add2vals'
            }
        }
    } catch (Exception e) {
        echo "Pipeline failed: ${e.message}"
        currentBuild.result = 'FAILURE'
    }
}
