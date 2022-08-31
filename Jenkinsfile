node(label: 'raspberrypi') {
    properties([
            disableConcurrentBuilds(),
            durabilityHint(hint: 'PERFORMANCE_OPTIMIZED')
        ])

    def srcdir = "${WORKSPACE}/src"
    def resultsdir = "results"

    // Checkout source code into a src directory
    stage("Checkout") {
        sh "rm -fr ${srcdir}"
        sh "mkdir ${srcdir}"
        dir(srcdir) {
            checkout scm
        }
    }

    // Build package and store into a results directory
    stage("Build") {
        sh "rm -fr ${resultsdir}"
        sh "mkdir -p ${resultsdir}"
        dir(srcdir) {
            sh "BRANCH=${env.BRANCH_NAME} pdebuild --use-pdebuild-internal --debbuildopts -b --buildresult ${WORKSPACE}/${resultsdir}"
        }
        archiveArtifacts artifacts: "${resultsdir}/*.deb", fingerprint: true
    }
}