node {
    stage('Build') {
        docker.image('python:3.12.1-alpine3.19').inside {
            sh 'python -m py_compile tree/master/sources/add2vals.py tree/master/sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml tree/master/sources/test_calc.py'
        }

        post {
            always {
                junit 'test-reports/results.xml'
            }
        }
    }
}


