pipeline {
    agent {
        kubernetes {
        label 'Gradle Builder'
        containerTemplate {
            name 'gradle'
            image 'gradle:6.3-jdk14'
            command 'sleep'
            args '30d'
        }
        podRetention onFailure()
        }
    }

     stages {      
          stage("Compile") {
              when {
                expression { GIT_BRANCH.indexOf('playground') == -1 }
              }
               steps {
                    sh "chmod +x gradlew"
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
              when {
                expression { GIT_BRANCH.indexOf('feature') > -1 }
              }
               steps {
                    sh "chmod +x gradlew"
                    sh "./gradlew test"
               }
          }
          stage("Code coverage") {
              when {
                expression { GIT_BRANCH.indexOf('main') > -1 }
              }
               steps {
                    sh "chmod +x gradlew"
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
               }
          }
          stage("Static code analysis") {
              when {
                expression { GIT_BRANCH.indexOf('feature') > -1 }
              }
               steps {
                    sh "chmod +x gradlew"
                    sh "./gradlew checkstyleMain"
               }
          }
          stage("Package") {
              when {
                expression { GIT_BRANCH.indexOf('feature') > -1 }
              }
               steps {
                    sh "chmod +x gradlew"
                    sh "./gradlew build"
               }
          }
     }
}