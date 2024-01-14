node {
    // Stage for building
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    // Stage for testing
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }

        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }

    // Stage for Manual Approval
    stage('Manual Approval') {
        agent any

        options {
            timeout(time: 1, unit: 'HOURS') // Set an appropriate timeout
        }

        steps {
            script {
                def userInput = input(
                    message: 'Lanjutkan ke tahap Deploy?',
                    ok: 'Proceed',
                    parameters: [booleanParam(defaultValue: true, description: 'Proceed to Deploy?', name: 'DEPLOY_APPROVAL')]
                )
                
                if (!userInput.DEPLOY_APPROVAL) {
                    error('Pipeline aborted by user')
                }
            }
        }
    }

    // Stage for deployment
    stage('Deploy') {
        when {
            expression { return env.DEPLOY_APPROVAL == 'true' }
        }

        agent any

        environment {
            VOLUME = '$(pwd)/sources:/src'
            IMAGE = 'cdrx/pyinstaller-linux:python2'
        }

        steps {
            dir(path: env.BUILD_ID) {
                unstash(name: 'compiled-results')
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                echo 'Jeda 1 menit saja...'
                sleep time: 60, unit: 'SECONDS'
            }
        }

        post {
            success {
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            }
        }
        echo 'Jeda 1 menit saja...'
        sleep time: 60, unit: 'SECONDS'
    }
}
