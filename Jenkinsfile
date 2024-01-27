node {
    try {
        // Build stage
        stage('Build') {
            def VOLUME = "${pwd()}/sources:/src"
            def IMAGE = 'python:2-alpine'

            // Use docker.image inside the script block
            script {
                docker.image(IMAGE).inside {
                    sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                    stash(name: 'compiled-results', includes: 'sources/*.py*')
                }
            }
        }

        // Test stage
        stage('Test') {
            def VOLUME = "${pwd()}/sources:/src"
            def IMAGE = 'qnib/pytest'

            // Use docker.image inside the script block
            script {
                docker.image(IMAGE).inside {
                    sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
                }
            }

            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        // Manual Approval stage
        stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap deploy?', ok: 'Proceed'
        }

        // Deploy stage
        stage('Deploy') {
            def VOLUME = "${pwd()}/sources:/src"
            def IMAGE = 'cdrx/pyinstaller-linux:python2'

            // Use docker.image inside the script block
            script {
                docker.image(IMAGE).inside {
                    dir(path: env.BUILD_ID) {
                        unstash(name: 'compiled-results')
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                    }
                }
            }
        }

        // Pause stage
        stage('Pause') {
            echo 'Pausing for 60 seconds...'
            sleep time: 60, unit: 'SECONDS'
        }

    } catch (Exception e) {
        // Handle errors and mark the build as unstable
        currentBuild.result = 'UNSTABLE'
        throw e
    }
}
