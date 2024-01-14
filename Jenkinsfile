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

    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?', ok: 'Lanjutkan'  
    }

    // Stage for deployment
    stage('Deploy') {
        environment {
            VOLUME = "${pwd()}/sources:/src"
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

        script {
            echo 'Jeda 1 menit saja...'
            sleep time: 60, unit: 'SECONDS'
        }
    }
}
