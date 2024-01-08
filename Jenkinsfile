node {
    try {
    
		environment {
			JENKINS_USER_NAME = "${sh(script:'id -un', returnStdout: true).trim()}"
			JENKINS_USER_ID = "${sh(script:'id -u', returnStdout: true).trim()}"
			JENKINS_GROUP_ID = "${sh(script:'id -g', returnStdout: true).trim()}"
		}
		agent {
			dockerfile {
				filename 'Dockerfile.build'
				additionalBuildArgs '''\
				--build-arg GID=$JENKINS_GROUP_ID \
				--build-arg UID=$JENKINS_USER_ID \
				--build-arg UNAME=$JENKINS_USER_NAME \
				'''
			}
		}

        // Build stage
        stage('Build') {
            docker.image('python:3.12.1-alpine3.19').inside {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
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
            docker.image('python:3.12.1-alpine3.19').inside {
                // Use the existing non-root user to install and run pyinstaller
                sh 'pip install --user pyinstaller'
                sh 'pyinstaller --onefile sources/add2vals.py'
                archiveArtifacts 'dist/add2vals'
            }
        }

    } catch (Exception e) {
        echo "Pipeline failed: ${e.message}"
        currentBuild.result = 'FAILURE'
    }
}
