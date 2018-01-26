# What are Flows?


## Fn server 와 Flow server 시작

터미널에서 functions server 를 실행한다. 기존에 실행 했으면 다음으로 넘어간다.:

~~~sh
$ fn start
~~~

비슷하게 Flows service 를 실행한다. 이때 API_URI 값을 fnserver로 설정한다.:

~~~sh
$ DOCKER_LOCALHOST=$(docker inspect --type container -f '{{.NetworkSettings.Gateway}}' fnserver)

$ docker run --rm  \
       -p 8081:8081 \
       -d \
       -e API_URL="http://$DOCKER_LOCALHOST:8080/r" \
       -e no_proxy=$DOCKER_LOCALHOST \
       --name flow-service \
       fnproject/flow:latest
~~~

Flow를 그래픽적으로 보기위한 flow UI 컨테이너도 필요하면 실행한다.:

~~~sh
$ docker run --rm \
       -p 3002:3000 \
       -d \
       --name flowui \
       -e API_URL=http://$DOCKER_LOCALHOST:8080 \
       -e COMPLETER_BASE_URL=http://$DOCKER_LOCALHOST:8081 \
       fnproject/flow:ui
~~~

## 첫번째 Fn Flow

### 첫번째 Flow 애플래케이션

Fn Java FDK를 이용하여 Maven 기반의 Java Function을 만든다. 자세한 것은 [Fn Project](./fn.md)을 읽어보기 바란다.

~~~sh 
$ mkdir example-flow-function && cd example-flow-function
$ fn init --runtime=java your_dockerhub_account/flow-primes
Runtime: java
function boilerplate generated.
func.yaml created
~~~

### Flow 만들기



~~~sh
mkdir text
cd text

mkdir grep
mkdir head
mkdir linecount
mkdir flow

vi app.yaml
app: text
~~~

### 1. grep function 만들기
~~~sh
$ cd grep
~~~

Dockerfile, func.sh, func.yaml 3개 파일을 만든다.

#### - Dockerfile
~~~dockerfile
FROM ubuntu:16.04
COPY func.sh /func.sh
CMD /func.sh
~~~

#### - func.sh 
~~~sh
#!/bin/bash
word=${FN_HEADER_Word:-foo}
grep -ie "$word" 
~~~

#### - func.yaml
~~~yaml
name: grep
~~~

func.sh 파일은 실행할 수 있어야 하므로 permission을 변경한다.
~~~sh
$ chmod +x func.sh
~~~




### 2. head function 만들기
~~~sh
$ cd head
~~~

Dockerfile, func.sh, func.yaml 3개 파일을 만든다.

#### - Dockerfile
~~~dockerfile
FROM ubuntu:16.04
COPY func.sh /func.sh
CMD /func.sh
~~~

#### - func.sh 
~~~sh
#!/bin/bash 
set -x 
lines=${FN_HEADER_Lines:-10}
(>&2 echo Taking first $lines from input )
head -n $lines
~~~

#### - func.yaml
~~~yaml
name: head
~~~

func.sh 파일은 실행할 수 있어야 하므로 permission을 변경한다.
~~~sh
$ chmod +x func.sh
~~~






### 3. linecount function 만들기
~~~sh
$ cd head
~~~

Dockerfile, func.sh, func.yaml 3개 파일을 만든다.

#### - Dockerfile
~~~dockerfile
FROM ubuntu:16.04
COPY func.sh /func.sh
CMD /func.sh
~~~

#### - func.sh 
~~~sh
#!/bin/bash 
wc -l 
~~~

#### - func.yaml
~~~yaml
name: linecount
~~~

func.sh 파일은 실행할 수 있어야 하므로 permission을 변경한다.
~~~sh
$ chmod +x func.sh
~~~



### 4. flow function 만들기

~~~sh
$ fn init --runtime java
~~~

function build 때 test 구문이 동작하기 때문에 test를 넘어가기 위해서 test 로직을 삭제한다.
~~~sh
$ rm -rf src/test
~~~

HelloFunction.java 를 다음과 같이 수정한다.
~~~java
package com.example.fn;

import com.fnproject.fn.api.flow.Flow;
import com.fnproject.fn.api.flow.FlowFuture;
import com.fnproject.fn.api.flow.Flows;
import com.fnproject.fn.api.flow.HttpResponse;

import static com.fnproject.fn.api.Headers.emptyHeaders;
import static com.fnproject.fn.api.flow.HttpMethod.POST;

public class HelloFunction {

    public String handleRequest(String input) {
        Flow flow = Flows.currentFlow();

        // 입력 값의 head 부분 저장
        FlowFuture<byte[]> headText = 
            flow.invokeFunction( "./head", 
                    POST, 
                    emptyHeaders(), 
                    input.getBytes())
                .thenApply(HttpResponse::getBodyAsBytes);

        // "Fn" 단어가 나온 부분을 검색하
        FlowFuture<byte[]> wordCountResult =
            flow.invokeFunction( "./grep", 
                    POST, 
                    emptyHeaders().withHeader("WORD", "Fn"),
                    input.getBytes())
                .thenApply(HttpResponse::getBodyAsBytes)
                // 검색된 단어가 몇개인지 계산하여 저장
                .thenCompose( grepResponse ->
                      flow.invokeFunction("./linecount", 
                              POST, 
                              emptyHeaders(), 
                              grepResponse ))
                .thenApply(HttpResponse::getBodyAsBytes);

        return "Number of times I found 'Fn': " + new String(wordCountResult.get()) + 
                "\nThe first head lines are: \n" + new String(headText.get());
    }
}
~~~


### 빌드

~~~sh
fn deploy --app text --all --local
DOCKER_LOCALHOST=$(docker inspect --type container -f '{{.NetworkSettings.Gateway}}' fnserver)
fn apps config set text COMPLETER_BASE_URL "http://$DOCKER_LOCALHOST:8081"

fn apps list

fn routes list text
~~~


### 실행
~~~sh
cd
git clone https://github.com/shiftyou/fnproject.git
curl --data-binary @./fnproject/fn.md http://localhost:8080/r/text/flow
~~~

flow ui를 통해서 호출되는 정보를 보려면
~~~http
http://localhost:3002
~~~

fn server 의 상태를 보려면
~~~sh
docker run --rm -it --link fnserver:api -p 4000:4000 -e "FN_API_URL=http:///localhost:8080" fnproject/ui
~~~