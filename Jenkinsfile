node {
    stage('Build') {
        checkout scm
        docker.image('python:2-alpine').inside {
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
        input message: 'Lanjutkan ke tahap deploy?', ok: 'Proceed'
    }

    stage('Deploy') {
        withEnv(["VOLUME=\$(pwd)/sources:/src", "IMAGE=cdrx/pyinstaller-linux:python2"]) {
            dir(env.BUILD_ID) {
                unstash(name: 'compiled-results')
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
            }
        }
    }

    stage('Pause') {
        echo 'Jeda 1 menit...'
        sleep time: 60, unit: 'SECONDS'
    }
}
