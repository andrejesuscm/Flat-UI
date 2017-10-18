#!/usr/bin/env groovy

pipeline {
  try {
    def TAG
    def REVISION
    def ID

    stage('Checkout') {
      deleteDir() // Workdir cleanup
      def scmVars = checkout scm
      // there is no way to configure PreBuildMerge in MultiBranch pipeline job, so we could use such workaround
      // https://support.cloudbees.com/hc/en-us/articles/226122247-How-to-Customize-Checkout-for-Pipeline-Multibranch-
      // but PreBuildMerge sets GIT_COMMIT variable to HEAD commit of the merged branch :(
      sh 'git merge -s recursive --no-ff origin/master'

      REVISION = scmVars.GIT_COMMIT
      ID = UUID.randomUUID().toString()[-12..-1]
      TAG = "${REVISION[0..7]}-${ID}"

      // Notifies the Stash Instance of an INPROGRESS build. It has to be executed after `checkout scm`, because
      // by default Jenkins will use the commit that was built by the Git plugin.
      step([$class: 'StashNotifier'])
    }

    stage("Tests") {
      def image = docker.build(TAG, "test/")
      image.inside("--cap-add=SYS_ADMIN") {

        // prepare
        sh 'npm install --silent'
        sh 'npm run lerna:bootstrap'

        // Run tests
        [
          'lint:js',
          'lint:css',
          'test:unit',
          'build:demos'
        ].each {
          sh "npm run ${it}"
        }

        junit "test-results.xml"

        publishHTML(target: [
          allowMissing: false,
          keepAll     : true,
          reportDir   : 'coverage',
          reportFiles : 'index.html',
          reportName  : 'Code coverage'
        ])

        publishHTML(target: [
          allowMissing: false,
          keepAll     : true,
          reportDir   : 'demos',
          reportFiles : 'index.html',
          reportName  : 'Demos'
        ])

        if (currentBuild.result == 'UNSTABLE') {
          throw new Exception('Build is unstable')
        }
      }
    }

    currentBuild.result = 'SUCCESS'
  } catch (e) {
    echo e.toString()
    currentBuild.result = 'FAILED'
  }
  finally {
    step([$class: 'StashNotifier'])
  }
}
