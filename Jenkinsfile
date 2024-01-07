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
				sh 'adduser -D -u 1010 myuser'  // Create a non-root user with UID 1000
				sh 'su myuser -c "pip install pyinstaller --user"'
				sh 'su myuser -c "pyinstaller --onefile sources/add2vals.py"'
				archiveArtifacts 'dist/add2vals'
			}
		}

    } catch (Exception e) {
        echo "Pipeline failed: ${e.message}"
        currentBuild.result = 'FAILURE'
    }
}
