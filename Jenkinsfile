node {
    try {
        stage('Build') {
            docker.image('python:2-alpine').inside {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }

        stage('Test') {
            docker.image('qnib/pytest').inside {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }

        stage('Deploy') {
            docker.image('cdrx/pyinstaller-linux:python2').inside {
                sh 'echo "Proses Deploy"'
                sh 'pyinstaller --onefile sources/add2vals.py'
            }
        }

        // Post-build actions
        junit 'test-reports/results.xml'

        // Archive artifacts on success
        success {
            archiveArtifacts 'dist/add2vals'
        }
    } catch (Exception e) {
        // Handle pipeline failure
        echo "Pipeline failed: ${e.message}"
        currentBuild.result = 'FAILURE'
    }
}
