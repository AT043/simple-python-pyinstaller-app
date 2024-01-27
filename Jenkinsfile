node {
    // Set options
    options {
        skipStagesAfterUnstable()
    }

    // Build stage
    stage('Build') {
        // Define Docker agent
        docker.image('python:2-alpine').inside {
            // Run build steps
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    // Test stage
    stage('Test') {
        // Define Docker agent
        docker.image('qnib/pytest').inside {
            // Run test steps
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        
        // Post-test steps
        junit 'test-reports/results.xml'
    }

    // Manual Approval stage
    stage('Manual Approval') {
        // Wait for manual approval
        input message: 'Lanjutkan ke tahap deploy?', ok: 'Proceed'
    }

    // Deploy stage
    stage('Deploy') {
        // Define any agent (any node)
        agent any
        
        // Set environment variables
        def VOLUME = "${pwd()}/sources:/src"
        def IMAGE = 'cdrx/pyinstaller-linux:python2'

        // Run deploy steps
        dir(path: env.BUILD_ID) {
            unstash(name: 'compiled-results')
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        }
    }

    // Pause stage
    stage('Pause') {
        // Pause for 60 seconds
        echo 'Pausing for 60 seconds...'
        sleep time: 60, unit: 'SECONDS'
    }
}
