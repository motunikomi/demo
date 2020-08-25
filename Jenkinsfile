pipeline {
  agent any
  stages {
    stage('build') {
      steps {
        sh 'chmod +x ./gradlew'
        sh './gradlew clean assemble'
      }
    }

    stage('test') {
      parallel {
        stage('test') {
          post {
            success {
              junit resultPath
              recordIssues(tool: checkStyle(pattern: checkstyleReport))
              recordIssues(tool: pmdParser(pattern: pmdReport))
              recordIssues(tool: spotBugs(pattern: spotbugsReport))
            }

          }
          steps {
            sh './gradlew check'
          }
        }

      }
    }

    stage('createWar') {
      steps {
        sh './gradlew bootWar'
        archiveArtifacts 'build/libs/*.war'
      }
    }
    stage('deploy') {
      steps {
        sshPublisher(
  publishers: [
    sshPublisherDesc(
      configName: 'ssh_motu',
      transfers: [
        sshTransfer(
          excludes: '',
          execCommand: '',
          execTimeout: 120000,
          flatten: false,
          makeEmptyDirs: false,
          noDefaultExcludes: false,
          patternSeparator: '[, ]+',
          remoteDirectory: '',
          remoteDirectorySDF: false,
          removePrefix: 'build/libs',
          sourceFiles: 'build/libs/*.war'
        )
      ],
      usePromotionTimestamp: false,
      useWorkspaceInPromotion: false,
      verbose: false
    )
  ]
)
      }
    }
  }
  environment {
    resultPath = 'build/test-results/**/TEST-*.xml'
    jacocoReportDir = 'build/jacoco'
    checkstyleReport = 'build/reports/checkstyle/*.xml'
    pmdReport = 'build/reports/pmd/*.xml'
    spotbugsReport = 'build/reports/spotbugs/*.xml'
  }
}