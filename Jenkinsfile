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
			docker.image('python:3.12.1-alpine3.19').inside('-u 1000') {
				// Set PIP_CACHE_DIR to a writable directory
				sh 'export PIP_CACHE_DIR="/tmp/pip_cache_dir"'
				
				// Install pyinstaller using pip with the specified cache directory
				sh 'pip install --cache-dir="/tmp/pip_cache_dir" pyinstaller'
				
				// Run pyinstaller
				sh 'pyinstaller --onefile sources/add2vals.py'
				
				// Archive artifacts
				archiveArtifacts 'dist/add2vals'
			}
		}


    } catch (Exception e) {
        echo "Pipeline failed: ${e.message}"
        currentBuild.result = 'FAILURE'
    }
}
