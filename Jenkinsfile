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

        stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap Deploy?', submitter: 'user'
        }

        stage('Deploy') {
            docker.image('python:3.12.1-alpine3.19').inside('-u 0:0') {
                // Uncomment and adjust the following lines based on your Linux distribution's package manager
                sh 'apk add --no-cache binutils'

                // Install pyinstaller without --user flag
                sh 'pip install pyinstaller'

                // Run pyinstaller
                sh 'pyinstaller --onefile sources/add2vals.py'
                archiveArtifacts 'dist/add2vals'
            }
        }

    } catch (Exception e) {
        echo "Pipeline failed: ${e.message}"
        currentBuild.result = 'FAILURE'
    }
}
