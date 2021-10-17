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
    echo 'Branch that triggered the job is : ' + scm.branches[0].name

    def myjobbranch = scm.branches[0].name
    def scmbranch = '';
    if(myjobbranch.indexOf('main') > -1){
      scmbranch = 'main';
    }else if(myjobbranch.indexOf('feature1') > -1){
      scmbranch = 'feature1'
    }else if(myjobbranch.indexOf('playground') > -1){
      scmbranch = 'playground'
    }
    
    container('gradle') {
      echo "myjobbranch value inside gradle is :" + myjobbranch
      echo "scmbranch value inside gradle is :" + scmbranch

      
      git url: 'https://github.com/kamalakar07/week6.git', branch: scmbranch

      stage("debug stage") {
          sh """
          echo ${env.BRANCH_NAME}
          echo ${scm.branches[0].name }
          """
      }       
      stage("Compile") {
          if(env.BRANCH_NAME != 'playground') {
              sh "chmod +x gradlew"
              sh "./gradlew compileJava"
          }
      }  
      stage("Unit test") {
          if(scmbranch == 'main' || scmbranch == 'feature1' ) {
              sh "chmod +x gradlew"
              sh "./gradlew test"
          }
      }
      stage("Code coverage") {
          if(scmbranch == 'main') {
              sh "chmod +x gradlew"
              sh "./gradlew jacocoTestReport"
              sh "./gradlew jacocoTestCoverageVerification"
          }
      } 
      stage("Static code analysis") {
          if(scmbranch == 'main' || scmbranch == 'feature1' ) {
              sh "chmod +x gradlew"
              sh "./gradlew checkstyleMain"
          }
      }  
      stage("Static code analysis") {
          if(scmbranch == 'main' || scmbranch == 'feature1' ) {
              sh "chmod +x gradlew"
              sh "./gradlew checkstyleMain"
          }
      }
      stage('Package') {
          sh '''
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
      }
    }

    container('kaniko') {
      stage('Build Image') {
        sh '''
        echo 'FROM openjdk:8-jre' > Dockerfile
        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
        /kaniko/executor --context `pwd` --destination kamalakar07/hello-kaniko:1.0
        '''
      }
    }

  }
}
