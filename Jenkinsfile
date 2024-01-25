pipeline {
    agent any

    // Stage for building
    stage('Build') {
        steps {
            script {
                docker.image('python:2-alpine').inside {
                    sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                    stash name: 'compiled-results', includes: 'sources/*.py*'
                }
            }
        }
    }

    // Stage for testing
    stage('Test') {
        steps {
            script {
                docker.image('qnib/pytest').inside {
                    sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
                }
            }
        }

        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }

    // Stage for manual approval
    stage('Manual Approval') {
        steps {
            script {
                input message: 'Lanjut ke tahap deploy?'
            }
        }
    }

	// Stage for deployment
	stage('Deploy') {
		steps {
			script {
				// Define environment variables
				def VOLUME = "${pwd()}/sources:/src"
				def IMAGE = 'cdrx/pyinstaller-linux:python2'

				// Deployment steps
				dir(path: env.BUILD_ID) {
					unstash name: 'compiled-results'
					sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
					archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
					sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
				}

				// Post-deployment actions
				post {
					success {
						echo 'Jeda 1 menit sebelum menjalankan aplikasi...'
						sleep time: 60, unit: 'SECONDS'
					}
				}
			}
		}
	}
}
