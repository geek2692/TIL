## Jenkins PipeLine

- 흐름 구성
- 각각의 동작 정의



### 등장 배경

- CI/CD 복잡해짐 -> 여러형태의 비슷한 job을 모듈화 -> 관리 및 유지보수 용이



#### Pipeline 2가지 방식

Scripted 방식

- Script 방식은 groovy 문법
- 자유도가 높음
- 기능은 좀 더 많음
- 구조가 없어서 손이 많이 감

Declarative

- Script 방식에 비해 코드 양이 적고 직관적



#### CI Tools

- Cloud에서 간단하게 이용하고 싶으면 Circle CI, Travis CI
- 위 방식은 github와 연동이 잘되어있음

Circle CI

- github와 연동을 해놓으면 소스변경이 있을때마다 자동으로 빌드

### Jenkins

- ### Jenkins Configuration as Code (jcasc)

  - 설정을 코드로 관리
  - 사내에서 표준화된 Jenkins 바이너리 제공을 목표로..

- ### CloudBees Jenkins Distribution

  - Open Source Jenkins와 차이점은 Jenkins Update나 Plugin Update 시 호환성 문제가 있다.
  - 버젼 관련해서 권장 가이드, 리포트를 제시해준다 (호환성)



### Jenkins Pipeline - 기본

#### 	Step
    -  대부분의 Plugin 동작들..
    -  Pipeline Syntax에서 대부분의 Step을 문법을 외울필요 없이 자동으로 생성할 수 있다.

#### 	기본 Step
- returnStdout: output을 반환

  ```groovy
  node {
      stage('s') {
          def output = sh encoding: 'UTF-8', returnStdout: true, script: 'java -version'
          echo output
  }}
  ```

- returnStatus: status code를 반환

  ```groovy
  node {
      stage('s') {
          def output = sh(encoding: 'UTF-8', returnStatus: true, script: 'java -version')
          echo output.toString()
  }}
  ```

위 두개를 한꺼번에 갖고 올 수 있는 방법은 아직 없음..



- label: 직관적으로 바로 알 수 있음(제목 붙이기)

  ```groovy
  node {
      stage('s') {
          def output = sh(encoding: 'UTF-8', 
                              label: 'print java version',  // set label
                              returnStatus: true, script: 'java -version')
          echo output.toString()
      }
  }
  ```

- dir

  ```groovy
  node {
      stage('s') {
          sh 'mkdir -p hello'
          sh 'pwd'
          dir('hello') {
              sh 'pwd'
  }}}
  ```



### stash, unstash

- node를 여러개 사용하는 경우 stash, unstash를 사용하게 되면 자동으로 옮겨지게 된다.
- binary를 archive 할때 사용할 수 있음



### basic example  (git checkout, excute)

```groovy
node {
    stage('git checkout') {   
        git credentialsId: 'github_hyunil-shin', url: 'https://github.com/hyunil-shin/JenkinsPipelineTraining'
    }

    stage('execute') {    
        dir('shell_scripts') {
            sh 'chmod +x ./script1.sh'
            sh './script1.sh'
        }
        
    }
}
```


#### 빌드 실패 및 관리방안

- 빌드 실패가 발생되면 다음 작업 중지

- try-catch로 성공-실패 제어

  ```groovy
  node {
      try {
          // do something that fails
          sh "exit 1"
      } catch (Exception err) {
         
      }
  }
  ```

- 강제실패

  ```groovy
  node() {    
      stage("s1") {
          sh 'ls -al'
      }
      
      def aa
      stage("s2") {        
          aa = sh script: 'exit 1', returnStatus: true        
          if(aa != 0) {
              error "hello"              // <=== 여기서 실행이 중지된다.
          }
      }
      
      stage("s3") {
          sh 'ls -al'
      }    
  }
  ```

- 특정 stage 실패

  - warnError, unstable step으로 stage 실패를 구분 할 수 있다.



### 종합 + mvn

- properties 매개변수: ${params.xxx}

- maven build

  ```groovy
  withEnv(["PATH+MAVEN=${tool 'mvn-3.6.0'}/bin"]) {
              sh 'mvn --version'
              sh "mvn clean ${params.testType}"
          }
  ```

- junit

  ```groovy
   stage('report') {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/**.exec'
      }
  ```

- ### emailtext

  ```groovy
  stage('send mail') {
          def mailRecipients = "david.park@gmail.co.kr"
          emailext attachLog: true, body: 'Jenkins Pipeline Tranining', 
                  subject: 'Welcome! TOAST FORWARD', 
                  to: mailRecipients
      }
  ```



### Jenkins API

- Jenkins 모든 데이터에 접근 할 수 있기때문에 기본적으로 API를 호출하면 다 막혀있음
- 어드민이 허용을 해줘야 사용가능
- 좀 더 많은것들을 할 수 있음(env, currentBuild ...)



## Jenkins Advanced

### Reusable Pipeline Libraries

- 라이브러리를 만들어놓고 함수 호출하는것처럼 사용할 수 있음

### Parallel

```groovy
node {
    Map builds = [:]
    List jdks = ["openjdk8", 'openjdk9', 'openjdk10']
    jdks.each {
        builds[it] = {
            stage(it) {
                dir(it) {
                    git 'https://github.com/hyunil-shin/java-maven-junit-helloworld.git'
                    def jdk = tool name: it, type: 'jdk'
                    withEnv(["JAVA_HOME=${jdk}", "PATH+JDK=${jdk}/bin", "PATH+MAVEN=${tool 'mvn-3.6.0'}/bin"]) {
                        sh "mvn clean test -Dsurefire.reportNameSuffix=${it}"
                        stash includes: 'target/surefire-reports/*.xml', name: it
                    }
                }
            }
        }
    }
    
    parallel builds
    
    stage('report') {
        dir('junit') {
            jdks.each {
                unstash it
            }
        }
        junit 'junit/**/*.xml'
    }
}
```



### replay

- pipeline replay는 빌드가 실패했을때 수정된 버젼으로 다시 실행할 수 있는 기능
- 커밋 전 스크립트가 옳바른지 확인 할 수 있는 방법 중 하나



#### 정리

- 노드를 여러개 쓸거면 노드를 먼저 정의
- 각 stage 구분
- 각 stage에서 해야될 동작들을 step을 가지고 정의
- 해결이 안될경우 groovy 문법을 이용해서 
- 위 방법들이 아닌경우 Jenkins API 호출
- 편하게 하고 싶다하면 라이브러리에 공통 함수 정의
- 그 외 병렬 실행등으로 실행을 더 빨리 할 수있고
- replay와 같은 방법으로 스크립트 검증
