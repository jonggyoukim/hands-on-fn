# Fn Project 가 무엇인가?
Fn Project는 4개의 주요 컴포넌트로 이루어져 있다.

1. Fn Server

    개발자가 자신의 기능을 멀티 클라우드 환경으로 쉽게 구축, 배포 및 확장 할 수 있도록 지원하는 기능형 서비스 시스템이다. 빠르고 신뢰할 수 있고 확장 가능하며 컨테이너 기반이다.

1. Fn Load Balancer

    운영자가 Fn 서버의 클러스터를 배치하고 트래픽을 인텔리전트하게 라우팅할 수 있다. 가장 중요한 점은 특정 기능에 대한 트래픽이 증가하면 부하를 분산할 뿐만 아니라 최적의 성능을 보장하기 위해 빨리 수행가능한 노드로 트래픽을 라우팅한다. 또한 전체 클러스터에 대한 정보를 수집하여 Fn 서버추가 또는 Fn 서버축소를 알 수 있다. 

1. Fn FDK

    Java 부터는 모든 언어로 함수를 신속하게 부트 스트랩하고, 함수 입력을 위한 데이터 바인딩 모델을 제공하고, 함수를 더 쉽게 테스트하고 기초를 마련하기 위한 목적으로 FDK 또는 Function Development Kit을 출시하였다.

1. Fn Flow

    개발자는 자신이 선택한 프로그래밍 언어 안에서 모든 기능의 상위 레벨 워크플로우를 구축하고 조율할 수 있다. 긴 json 또는 yaml 템플릿으로 구축된 복잡한 외부 모델을 배우지 않고도 병렬처리, 시퀀싱/체인화, 오류처리, 입출력 등을 쉽게 사용할 수 있다. 무엇보다다도 Flow 는 모든 함수 호출 그래프를 추적하여 대시보드에서 시각화, 전체 Flows의 전체 스택 로그 및 전체 기능 그래프에서의 변수/메모리 재구성을 허용한다.
    

# Fn Project 아키텍처
Fn Project의 전체적인 아키텍처는 다음과 같다.

![FN Architecture Diagram](https://github.com/fnproject/fn/blob/master/docs/assets/architecture.png?raw=true)

- Load Balancer

    어떠한 로드밸런스도 동작한다. Fn의 인스턴스 한테 로드밸런스를 통해서 요청이 가도록 한다.

- Database

    작은 데이터베이스를 사용하고 있다. request/response 시에 어떠한 것도 쓰여지지 않으며 쉽게 플러거블 하다.[found here](databases/README.md). 

- Message Queue

    메시지 큐는 리소스가 가능할 때 까지 프로세싱을 위해 버퍼링을 하기위해  비동기 호출에서 가장 중요한 부분을 차지 한다. 

    메시지 역시 플러거블 하며 현재는 몇가지의 옵션만을 제공한다.[found here](mqs/README.md). 

- Logging, Metrics and Monitoring

    로깅 또한 매우 중요한 부분이다.  [logspout-statsd](https://github.com/treeder/logspout-statsd)를 사용하여 로그로 부터 매트릭스를 수집할 수도 있다.
[More about Metrics](metrics.md)

- Scaling

    로그에 메트릭스가 생성되어 스케일링 할 때 사용자에게 알릴 수 있다. 대부분의 중요한 것은 동기와 비동기 functions 모두에게서 중요한 것은 `wait_time` 매트릭이다. 만약 `wait_time` 이 증가되면 더 많은 Fn 인스턴스를 시작해야 한다.


# Fn 설치
## 요구사항
- Docker 17.05 이상
- Docker Hub account 

## 다운로드
- Homebrew - MacOS

    ~~~sh
    brew install fn
    ~~~
- Shell script - Linux/MacOS

    ~~~sh
    curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
    ~~~
  
- Windows

    https://github.com/fnproject/cli/releases
    
# fn
다운받은 fn 을 수행하면 다음과 같이 나온다.
~~~sh
$ fn
~~~

~~~txt
fn 0.4.36

Fn command line tool

ENVIRONMENT VARIABLES:
   FN_API_URL - Fn server address
   FN_REGISTRY - Docker registry to push images to, use username only to push to Docker Hub - [[registry.hub.docker.com/]USERNAME]

COMMANDS:
     start         start a functions server
     update        pulls latest functions server
     init          create a local func.yaml file
     apps          manage applications
     routes        manage routes
     images        manage function images
     lambda        create and publish lambda functions
     version       displays cli and server versions
     calls         manage function calls for apps
     logs          manage function calls for apps
     test          run functions test if present
     build-server  build custom fn server
     help, h       Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --verbose --verbose, -v --verbose  Use --verbose to enable verbose mode for debugging
   --help, -h                         show help
   --version                          print only the version

LEARN MORE:
   https://github.com/fnproject/fn

~~~
명령어 중 몇가지를 보면 다음과 같다.

- start : Function App을 등록하여 운영하는 서버를 실행 (fn start)
- init : 새로운 Function App을 생성 (fn init --runtime java)
- update : fn Project 의 최신버젼으로 업데이트
- apps : 디플로이된 fn application 관리 (fn apps lsit)
- routes : 호출할 application 라우터 관리 (fn routes list hello)
- images : application을 관리 (fn images build / fn images push)
- calls : 호출된 application 기록 보기 (fn calls list hello)
- logs : fn application의 로그 보기 (fn logs get hello 01C4Q8E0WT47WWW00000000000)
- test : fn application을 테스트 (fn test)


# 첫번째 Function 프로그램

프로그래밍의 가장 기본인 "hello world" 부터 시작해 보도록 한다. 첫번째 언어를 go 언어 수행해 보겠는데, go 언어를 모르더라도 무서워 할 필요없다.
Function Programming 은 비지니스 로직에만 집중하기 때문에 go 언어를 위한 개발환경을 구성할 필요는 없다. Fn 이 Docker를 통해서 필요한 라이브러리와 환경을 구성해 줄 것이다.

## Docker Registry 연결

시작하기 전에 우리는 환경변수 하나를 설정해야 한다. RN_REGISTRY 라는 환경변수이며 Docker Registry를 위한 환경변수이다.  일반적으로 Docker Hub의 username을 사용한다. username을 fndemouser 라고 한다면 다음과 같이 설정한다. `로컬에서 실행만 할 예정이라면 설정하지 않아도 된다.`

~~~sh
$ export FN_REGISTRY=fndemouser
~~~
이제 시작해 보기로 한다.


## Function 만들기

hello 라는 디렉토리를 만든다.
~~~bash
$ mkdir hello
$ cd hello
~~~

다음으로 샘플을 만든다.
~~~sh
$ fn init --runtime go
~~~

--runtime 의 파라메터로 쓰일 수 있는 언어에 관한 파라메터는 dotnet, go, java8, java9, java, lambda-nodejs4.3, lambda-node-4, node, php, python, python2.7, python3.6, ruby, rust 이다.

샘플로 만들어진 소스인 func.go는 다음과 같다.
~~~java
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Person struct {
	Name string
}

func main() {
	p := &Person{Name: "World"}
	json.NewDecoder(os.Stdin).Decode(p)
	mapD := map[string]string{"message": fmt.Sprintf("Hello %s", p.Name)}
	mapB, _ := json.Marshal(mapD)
	fmt.Println(string(mapB))
}
~~~

이를 위한 yaml 파일인 func.yaml 은 다음과 같다.
~~~ yaml
version: 0.0.1
runtime: go
entrypoint: ./func
~~~
yaml 파일에는 다음을 포함한다.
- version : 버젼, 0.0.1 로 부터 시작한다.
- runtime : 실행환경의 이름 (go, java 등등)
- entrypoint : function이 불려질 때의 이름

## Function 실행하기

~~~sh
$ fn run
or
$ fn -verbose run
~~~

~~~log
Building image hello:0.0.1 
Sending build context to Docker daemon   5.12kB
Step 1/8 : FROM fnproject/go:dev as build-stage
dev: Pulling from fnproject/go
1160f4abea84: Pull complete 
4d49542c61a4: Pull complete 
3ee103c86f60: Pull complete 
9a56d2eb1eed: Pull complete 
67e0ebed9a3b: Pull complete 
194910951a14: Pull complete 
561c26385b71: Pull complete 
Digest: sha256:5535f7186fdd485a3b243b6909692b61c4b7f9bf6b2b5b1ea85a956ebc02141b
Status: Downloaded newer image for fnproject/go:dev
 ---> 1b2c8e7b6de5
Step 2/8 : WORKDIR /function
Removing intermediate container 45c1ed73593d
 ---> 53821cf80683
Step 3/8 : ADD . /go/src/func/
 ---> 93e6e47c1794
Step 4/8 : RUN cd /go/src/func/ && go build -o func
 ---> Running in 8756047555d0
Removing intermediate container 8756047555d0
 ---> 10cbcc9df704
Step 5/8 : FROM fnproject/go
latest: Pulling from fnproject/go
98b11b540f87: Pull complete 
4cce13e82d12: Pull complete 
Digest: sha256:e089485ddf57ae94cfcbbc4705a1ae23229e3ccd7f337e29ace4000c5da4e7f1
Status: Downloaded newer image for fnproject/go:latest
 ---> ec8378042107
Step 6/8 : WORKDIR /function
Removing intermediate container 8fb0194d96b5
 ---> 73f6f3b4dece
Step 7/8 : COPY --from=build-stage /go/src/func/func /function/
 ---> 505c176a5094
Step 8/8 : ENTRYPOINT ["./func"]
 ---> Running in 72c8966a321a
Removing intermediate container 72c8966a321a
 ---> d8684c01d340
Successfully built d8684c01d340
Successfully tagged hello:0.0.1

{"message":"Hello World"}
~~~

docker 에서 다운받은 이미지를 보면 다음과 같다.
~~~sh
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello               0.0.1               d8684c01d340        37 minutes ago      6.94MB
<none>              <none>              10cbcc9df704        38 minutes ago      517MB
fnproject/go        dev                 1b2c8e7b6de5        5 weeks ago         515MB
fnproject/go        latest              ec8378042107        5 weeks ago         4.7MB
~~~

### Function 테스트

func.json 파일에는 테스트 할 내용들이 들어있다.
~~~json
{
    "tests": [
        {
            "input": {
                "body": {
                    "name": "Johnny"
                }
            },
            "output": {
                "body": {
                    "message": "Hello Johnny"
                }
            }
        },
        {
            "input": {
                "body": ""
            },
            "output": {
                "body": {
                    "message": "Hello World"
                }
            }
        }
    ]
}
~~~


~~~
fn test
~~~
위와 같이 테스트를 실행하면 현재 2개의 테스트 항목이 있으며 이를 수행한다.
~~~
Building image hello:0.0.1 .
Running 2 tests on /home/shiftyou/fn/hello/func.yaml (image: hello:0.0.1):
Test 1
PASSED -    ( 707.213142ms )

Test 2
PASSED -    ( 759.404196ms )

tests run: 2 passed, 0 failed
~~~

`fn test`는 function application을 테스트 하는 것이며 아직 서비스로 등록이 되지 않았다. 서비스로 등록하기 위해서는 fn server 를 시작해야 한다.

  
## Fn Server 시작
~~~sh
fn start
~~~
실행을 하면, docker fn server 에 대한 docker image를 다운로드 받고 Container를 시작한다.
~~~
can't create unix socket /var/run/docker.sock: device or resource busy
time="2018-01-16T07:00:29Z" level=info msg="Setting log level to" level=info
time="2018-01-16T07:01:09Z" level=info msg="started tracer" url=
time="2018-01-16T07:01:09Z" level=info msg="datastore dialed" datastore=sqlite3 max_idle_connections=256
time="2018-01-16T07:01:09Z" level=info msg="no docker auths from config files found (this is fine)" error="open /root/.dockercfg: no such file or directory"
time="2018-01-16T07:01:09Z" level=info msg="available memory" availMemory=6781615719 cgroupLimit=9223372036854771712 headRoom=753512857 totalMemory=7535128576
time="2018-01-16T07:01:09Z" level=info msg="sync and async ram reservations" ramAsync=5425292576 ramAsyncHWMark=4340234060 ramSync=1356323143
time="2018-01-16T07:01:09Z" level=info msg="available cpu" availCPU=2000 totalCPU=2000
time="2018-01-16T07:01:09Z" level=info msg="sync and async cpu reservations" cpuAsync=1600 cpuAsyncHWMark=1280 cpuSync=400
time="2018-01-16T07:01:09Z" level=info msg="Fn serving on `:8080`" type=full

        ______
       / ____/___
      / /_  / __ \
     / __/ / / / /
    /_/   /_/ /_/
        v0.3.298

~~~

현재 등록되어 있는 Fn application 은 다음과 같이 확인 할 수 있다.
~~~sh
$ fn apps list
~~~

## Function 등록

Fn application을 수행하려면 Fn server에 등록해야 한다. 
~~~sh
$ fn deploy --app hello 
~~~
위의 명령은 application image를 Docker Hub에 등록을 하고 fn server에 배포하는 명령어이다.

Docker Hub에 등록하지 않고 fn server에만 배포하려면 다음과 같이 명령한다.
~~~
$ fn deploy --app hello --local
~~~

fn server에 등록된 application을 살펴볼 수 있다.
~~~sh
$ fn apps list
~~~

## Function 호출

fn server에 등록된 function을 직접 호출할 때에는 다음과 같이 호출한다.

~~~
$ fn call hello /hello
{"message":"Hello World"}

$ curl http://localhost:8080/r/myapp/hello
{"message":"Hello World"}
~~~

혹은 브라우저에서  http://localhost:8080/r/myapp/hello 를 호출하면 된다.

Function application을 호출하기 위한 endpoint 는 다음과 같이 알 수 있다.

~~~sh
$ fn routes list hello
path	image		endpoint
/hello	hello:0.0.3	localhost:8080/r/hello/hello
~~~

사용자의 데이터를 입력하기 위해 전송할 sample.json 파일을 만든다.
~~~json
{
    "name": "JongGyou"
}
~~~

input을 포함하여 호출을 해 본다.
~~~sh
$ cat sample.json | fn call hello /hello
{"message":"Hello JongGyou"}

$ curl -d @sample.json http://localhost:8080/r/hello/hello
{"message":"Hello JongGyou"}
~~~

---

# One application & Two function

지금까지 구성한 application은 하나의 application 에 하나의 function으로 구성이 된다. 이제 하나의 application에 두개의 function으로 구성하는 방법을 알아본다.




## 첫번째 Function 생성

Fn Project는 기본적으로 Java9을 지원하는 Developer Kit을 제공한다. 첫번째 function 으로 java로 구성된 function을 생성한다.

기존에 Fn application을 만드는 방법과 같이 --runtime 옵션의 parameter로 `java`를 지정해 주면 된다. 
(parameter 값은 dotnet, go, java8, java9, java, lambda-nodejs4.3, lambda-node-4, node, php, python, python2.7, python3.6, ruby, rust 이 가능하다.)

~~~sh
$ mkdir myapp; cd myapp
$ mkdir app1; cd app1
$ fn init --runtime java
Runtime: java
Function boilerplate generated.
func.yaml created.
~~~

만들어진 환경은 maven 기반의 개발환경으로 만들어지므로 pom.xml 이 존재한다.

~~~sh
$ ls
func.yaml  pom.xml  src
~~~

자동으로 만들어지는 소스는 다음과 같다.
~~~java
package com.example.fn;

public class HelloFunction {

    public String handleRequest(String input) {
        String name = (input == null || input.isEmpty()) ? "world"  : input;

        return "Hello, " + name + "!";
    }

}
~~~

실행하면 다음과 같이 결과가 나온다.
~~~sh
$ fn run
Building image app1:0.0.1 
Hello, world!
~~~

사용자의 input을 받으면 다음과 같이 `!`문자가 다음 줄칸에 나온다.

~~~sh
$ echo JongGyou | fn run
Building image app1:0.0.1 
Hello, JongGyou
!
~~~
이를 수정하기 위해서 수정한다.
~~~java
return "Hello, " + name + "!"; 구문을
return "Hello, " + name.trim() + "!"; 으로 수정
~~~

## 두번째 Function 생성
두번째 function은 node 로 구성하도록 한다.

~~~sh
$ fn init --runtime node
Runtime: node
func.yaml created.
~~~

수행할 func.js 를 다음과 같은 내용으로 만든다.
~~~node
name = "World";
fs = require('fs');
try {
	obj = JSON.parse(fs.readFileSync('/dev/stdin').toString())
	if (obj.name != "") {
		name = obj.name
	}
} catch(e) {}
console.log("Hello", name, "from Node!");
~~~

저장 후 테스트를 한다.
~~~sh
$ fn run
Building image app2:0.0.1 
Hello World from Node!
~~~



테스트에 사용할 test.json 파일은 다음과 같다.
~~~json
{
    "tests": [
        {
            "input": {
                "body": {
                    "name": "Johnny"
                }
            },
            "output": {
                "body": {
                    "message": "Hello Johnny from Node!"
                }
            }
        },
        {
            "input": {
                "body": ""
            },
            "output": {
                "body": {
                    "message": "Hello World from Node!"
                }
            }
        }
    ]
}
~~~



테스트를 수행하면 다음과 같다.
~~~sh
$ fn test
Building image app2:0.0.3 
Running 2 tests on /home/shiftyou/fn/myapp/app2/func.yaml (image: app2:0.0.3):
Test 1
PASSED -    ( 1.381575175s )

Test 2
PASSED -    ( 1.468638162s )

tests run: 2 passed, 0 failed
~~~

사용자가 입력할 sample.json 파일은 다음과 같다.
~~~json
{
    "name": "JongGyou"
}
~~~

사용자의 입력을 함께 수행하면 다음과 같다.
~~~sh
$ cat sample.jon | fn run
Building image app2:0.0.1 
Hello JongGyou from Node!
~~~


## Application 배포

두개의 function을 가진 application을 배포하기 위해서 app.yaml이 필요하다. myapp/app.yaml 을 다음의 내용으로 만든다.
~~~yaml
$ cat myapp.yaml
name: myapp
~~~

전체 구성되는 디렉토리는 다음과 같다.
~~~
myapp
├── app1
│   ├── func.yaml
│   ├── pom.xml
│   └── src
│       ├── main
│       │   └── java
│       │       └── com
│       │           └── example
│       │               └── fn
│       │                   └── HelloFunction.java
│       └── test
│           └── java
│               └── com
│                   └── example
│                       └── fn
│                           └── HelloFunctionTest.java
├── app2
│   ├── func.js
│   ├── func.yaml
│   ├── sample.json
│   └── test.json
└── app.yaml

13 directories, 9 files
~~~

application을 배포한다.
~~~sh
$ fn deploy --all --local
Deploying app1 to app: myapp at path: /app1
Bumped to version 0.0.3
Building image app1:0.0.3 
Updating route /app1 using image app1:0.0.3...
Deploying app2 to app: myapp at path: /app2
Bumped to version 0.0.2
Building image app2:0.0.2 
Updating route /app2 using image app2:0.0.2...
~~~

application 이름은 myapp, 두개의 function은 /app1, /app2 로 배포됨을 알 수 있다.

배포된 application을 체크해 본다.
~~~sh
$ fn apps l
~~~

배포된 application의 route를 체크해 본다.
~~~sh
$ fn routes l myapp
path	image		endpoint
/app1	app1:0.0.4	localhost:8080/r/myapp/app1
/app2	app2:0.0.3	localhost:8080/r/myapp/app2
~~~

각각의 function을 호출해 본다.
~~~sh
$ curl -d "JongGyou" http://localhost:8080/r/myapp/app1
Hello, JongGyou!

$ curl -d @sample.json http://localhost:8080/r/myapp/app2
Hello JongGyou from Node!
~~~

위와같이 하나의 application 이 가지고 있는 function이 여러개 있을 수 있다. 

function 간의 호출 흐름을 제어하기 위해서는 [Function Flow](./fnflow.md)를 살펴보기 바란다.




---
# 참고

fn 명령어의 name 파라메터에 대한 결과는 다음을 참조하면 잘 알수 있다.
~~~sh
$ mkdir aaa; cd aaa
$ fn init --runtime go --name bbb
$ fn deploy --app ccc --local
$ fn routes list ccc
path	image		endpoint
/aaa	bbb:0.0.1	localhost:8080/r/ccc/aaa
~~~
