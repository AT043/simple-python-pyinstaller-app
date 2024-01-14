node {
    // Skip stages after an unstable build
    options {
        skipStagesAfterUnstable()
    }

    // Stage for building
    stage('Build') {
        // Run in a Docker container with Python 2
        docker.image('python:2-alpine').inside {
            // Compile Python files and stash the results
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    // Stage for testing
    stage('Test') {
        // Run in a Docker container with qnib/pytest
        docker.image('qnib/pytest').inside {
            // Run pytest and store results in JUnit format
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }

        // Post-action for test stage
        post {
            always {
                // Archive JUnit test results
                junit 'test-reports/results.xml'
            }
        }
    }
    
    //stage for Manual Approval
    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed' 
    }

    // Stage for deployment
    stage('Deploy') {
        // Run on any available agent
        agent any

        // Define environment variables
        environment {
            VOLUME = '$(pwd)/sources:/src'
            IMAGE = 'cdrx/pyinstaller-linux:python2'
        }

        steps {
            // Inside the build directory
            dir(path: env.BUILD_ID) {
                // Unstash compiled results
                unstash(name: 'compiled-results')

                // Run pyinstaller in a Docker container
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
				
				// Echo and sleep after pyinstaller command
				echo 'Jeda 1 menit saja...'
				sleep time: 60, unit: 'SECONDS'
            
            }
        }

        // Post-action for deploy stage
        post {
            success {
                // Archive the deployed artifact
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"

                // Clean up build and dist directories
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            }
        }
        echo 'Jeda 1 menit saja...'
		sleep time: 60, unit: 'SECONDS'  
    }
}
