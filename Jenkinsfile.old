podTemplate(containers: [
    containerTemplate(
        name: 'gradle',
        image: 'gradle:6.3-jdk14',
        command: 'sleep',
        args: '30d'
    ),
]) {
node(POD_LABEL) {
     stage('Run pipeline against a gradle project') {
               container('gradle') {
               
                    stage("Compile") {
                      sh " ls -la"    
                      sh "cd week6"
                      sh "chmod +x gradlew"
                      sh "./gradlew compileJava"
                    }
                    stage("Unit test") {
                        sh "./gradlew test"
                    }
                    stage("Code coverage") {
                      sh "./gradlew jacocoTestReport"
                      sh "./gradlew jacocoTestCoverageVerification"
                    }
                    stage("Static code analysis") {
                        sh "./gradlew checkstyleMain"
                    }
                    stage("Package") {
                        sh "./gradlew build"
                    }

                    stage("Docker build") {
                        sh "docker build -t leszko/calculator:${BUILD_TIMESTAMP} ."
                    }

                    stage("Docker login") {
                      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub-credentials',
                                usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                           sh "docker login --username $USERNAME --password $PASSWORD"
                      }

                    }

                    stage("Docker push") {
                        sh "docker push leszko/calculator:${BUILD_TIMESTAMP}"

                    }

                    stage("Update version") {
                        sh "sed  -i 's/{{VERSION}}/${BUILD_TIMESTAMP}/g' calculator.yaml"
                    }
                    
                    stage("Deploy to staging") {
                      sh "kubectl config use-context staging"
                      sh "kubectl apply -f hazelcast.yaml"
                      sh "kubectl apply -f calculator.yaml"
                    }

                    stage("Acceptance test") {
                      sleep 60
                      sh "chmod +x acceptance-test.sh && ./acceptance-test.sh"
                    }

                    stage("Release") {
                      sh "kubectl config use-context production"
                      sh "kubectl apply -f hazelcast.yaml"
                      sh "kubectl apply -f calculator.yaml"
                    }
                    stage("Smoke test") {
                         sleep 60
                         sh "chmod +x smoke-test.sh && ./smoke-test.sh"
                    }

               }
    }
}
}