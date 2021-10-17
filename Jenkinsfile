podTemplate(yaml: '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: gradle
    image: gradle:6.3-jdk14
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
    - name: shared-storage
      mountPath: /mnt
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - sleep
    args:
    - 9999999
    volumeMounts:
    - name: shared-storage
      mountPath: /mnt
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: shared-storage
    persistentVolumeClaim:
      claimName: jenkins-pv-claim
  - name: kaniko-secret
    secret:
      secretName: dockercred
      items:
      - key: .dockerconfigjson
        path: config.json
''') {
  node(POD_LABEL) {
    echo 'Branch that triggered the job (env.BRANCH_NAME) is : ' + env.BRANCH_NAME
    
    container('gradle') {
      echo "env.BRANCH_NAME value inside gradle container is :" + env.BRANCH_NAME
      
      git url: 'https://github.com/kamalakar07/week6.git', branch: env.BRANCH_NAME

      if( 1 == 2){
        stage("debug stage") {
            sh """
            echo ${env.BRANCH_NAME}
            echo ${scm.branches[0].name }
            """
        } 
      }  
      if(env.BRANCH_NAME != 'playground') {    
        stage("Compile") {
          sh "chmod +x gradlew"
          sh "./gradlew compileJava"
        }
      }
      if(env.BRANCH_NAME == 'main' || env.BRANCH_NAME.indexOf("feature") > -1 ) {
        stage("Unit test") {
            sh "chmod +x gradlew"
            sh "./gradlew test"
        }
      }
      if(env.BRANCH_NAME == 'main') {
        stage("Code coverage") {         
          sh "chmod +x gradlew"
          sh "./gradlew jacocoTestReport"
          sh "./gradlew jacocoTestCoverageVerification"
        }
      } 
      if(env.BRANCH_NAME == 'main' || env.BRANCH_NAME.indexOf("feature") > -1 ) {
        stage("Static code analysis") {         
            sh "chmod +x gradlew"
            sh "./gradlew checkstyleMain"
        }
      }  
      if(env.BRANCH_NAME  == 'main' || env.BRANCH_NAME.indexOf("feature") > -1 ) {
        stage("Static code analysis") {
          sh "chmod +x gradlew"
          sh "./gradlew checkstyleMain"
        }
      }
      if(env.BRANCH_NAME  == 'main' || env.BRANCH_NAME.indexOf("feature") > -1 ) { 
        stage('Package') {
            sh '''
            chmod +x gradlew
            ./gradlew build
            mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
            '''
        }
      }
    }

    container('kaniko') {
      if(env.BRANCH_NAME != 'playground') { 
        stage('Build Image') {
          sh '''
          echo 'FROM openjdk:8-jre' > Dockerfile
          echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
          echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
          mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
          '''
          if(env.BRANCH_NAME  == 'main') { 
            sh '/kaniko/executor --context `pwd` --destination kamalakar07/calculator:1.0'
          }

          if(env.BRANCH_NAME.indexOf("feature") > -1 ) { 
            sh '/kaniko/executor --context `pwd` --destination kamalakar07/calculator-feature:0.1'
          }
        }
      }
    }

  }
}
