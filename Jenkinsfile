node(label: 'raspberrypi') {
    properties([
            disableConcurrentBuilds(),
            durabilityHint(hint: 'PERFORMANCE_OPTIMIZED')
        ])

    def srcdir = "${WORKSPACE}/src"
    def resultsdir = "results"

    // This one is a little different to the usual boilerplate, because we
    // want to build a single arch-independent package that works on multiple
    // distribution versions, not one per version. We build once on
    // build_dist / build_arch and then test-install against dist_and_arch_list

    def build_dist = "buster"
    def build_arch = "armhf"

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
            sh "DIST=${build_dist} ARCH=${build_arch} BRANCH=${env.BRANCH_NAME} pdebuild --use-pdebuild-internal --debbuildopts -b --buildresult ${WORKSPACE}/${resultsdir}"
        }
        archiveArtifacts artifacts: "${resultsdir}/*.deb", fingerprint: true
    }

    def dist_arch_list = [
      ["bookworm", "armhf"],
      ["bookworm", "arm64"],
      ["bullseye", "armhf"],
      ["bullseye", "arm64"],
      ["buster", "armhf"]
    ]

    for (int i = 0; i < dist_arch_list.size(); ++i) {
        def dist_and_arch = dist_arch_list[i]
        def dist = dist_and_arch[0]
        def arch = dist_and_arch[1]

        stage("Test install on ${dist} (${arch})") {
            sh "BRANCH=${env.BRANCH_NAME} ARCH=${arch} /build/pi-builder/scripts/validate-packages.sh ${dist} ${resultsdir}/flightaware-apt-repository_*.deb"
        }
    }

    // Deploy to all distribution repositories
    stage('Deploy to internal repository') {
        for (int i = 0; i < dist_arch_list.size(); ++i) {
            def dist_and_arch = dist_arch_list[i]
            def dist = dist_and_arch[0]
            def arch = dist_and_arch[1]
            sh "/build/pi-builder/scripts/deploy.sh -distribution ${dist} -architectures ${arch} -branch ${env.BRANCH_NAME} ${resultsdir}/*.deb"
        }
    }
}
