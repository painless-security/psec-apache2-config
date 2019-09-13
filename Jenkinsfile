pipeline {
  // Select build agent. This requires label 'debian'.
  agent { label 'debian' }
  
  // Build options
  options {
    checkoutToSubdirectory('psec-apache2-config')
  }
  
  // Sets environment for shell commands; available to Groovy script in env.*
  environment {
    DISTRIBUTION = 'buster'
    ARCH = 'amd64'

    // n.b., the substitutions in the next lines are in Groovy, not shell script.
    BASEPATH = "/var/cache/pbuilder/base-${env.DISTRIBUTION}-${env.ARCH}.cow"
    BUILDRESULT = "${env.WORKSPACE}/result"
  }
  
  // Define build stages. These run sequentially.
  stages {
    stage('Initialization') {
      steps {
        sh label: 'Update build environment', script: 'sudo cowbuilder update --basepath="${BASEPATH}"'
        
        // Without this, pdebuild creates the result directory as root, which
        // prevents subsequent builds from cleaning the workspace.
        sh label: 'Create result directory', script: 'mkdir -p "${BUILDRESULT}"'
      }
    }
    
    stage('Build') {
      steps {
        dir('psec-apache2-config') {
          sh label: 'Remove git files', script: 'rm -rf .git .gitignore'

          sh label: 'Tag build in changelog',
             script: '''dch --local "+${BUILD_NUMBER}~" \
                            --distribution "unstable" \
                            "Build ${BUILD_NUMBER}"'''

          sh label: 'Build package',
             script: '''pdebuild --pbuilder cowbuilder \
                                 --buildresult "${BUILDRESULT}" \
                                 -- \
                                 --basepath "${BASEPATH}"'''
        }
      }
    }
  }
  
  // Steps to run following a build
  post {
    // Run only after successful build
    success {
      archiveArtifacts artifacts: 'result/*'
    }
  }
}
