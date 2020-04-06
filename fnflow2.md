---
title: "Fnflow"
date: 2018-01-24T16:19:08+09:00
draft: false
---
# Fn Flow  User Guide

By following this step-by-step guide you will learn to create, run and deploy
a simple Java app in Fn that leverages the Fn Flow asynchronous execution
APIs.


## What are Flows?

Fn Flow consists of a set of client-side APIs for you to use within your
Fn apps, as well as a long-running server component (the _flow service_) that
orchestrates computation beyond the life-cycle of your functions. Together,
these components enable non-blocking asynchronous execution flows, where your
function only runs when it has useful work to perform. If you have used the Java
8 [CompletionStage](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/CompletionStage.html)
and
[CompletableFuture](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/CompletableFuture.html)
APIs, a lot of the concepts will already be familiar to you.

[Advanced Topics](FnFlowsAdvancedTopics.md) provides more detail on how data serialization and error handling works with Fn Flow under the covers.

## Pre-requisites
Before you get started, you will need to be familiar with the [Fn Java
FDK](../README.md) and have the following things:

* [Fn CLI](https://github.com/fnproject/cli)
* [Fn Java FDK](https://github.com/fnproject/fn-java-sdk)
* [Fn flow](https://github.com/fnproject/flow)
* [Docker-ce 17.06+ installed locally](https://docs.docker.com/engine/installation/)
* A [Docker Hub](http://hub.docker.com) account

### Install the Fn CLI tool

To install the Fn CLI tool, just run the following:

```
$ curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
```

This will download a shell script and execute it. If the script asks for
a password, that is because it invokes sudo.

### DockerHub 에 로그인

Docker Hub 어카운트로 로그인을 한다.

```
$ docker login
```

### 로컬 Fn server 와 Flow server 시작

터미널에서 functions server 를 실행한다.:

```
$ fn start
```

비슷하게 Flows server 를 실행한다. 이때 API_URI 값을 fnserver로 설정한다.:

```
$ DOCKER_LOCALHOST=$(docker inspect --type container -f '{{.NetworkSettings.Gateway}}' fnserver)

$ docker run --rm  \
       -p 8081:8081 \
       -d \
       -e API_URL="http://$DOCKER_LOCALHOST:8080/r" \
       -e no_proxy=$DOCKER_LOCALHOST \
       --name flow-service \
       fnproject/flow:latest
```

Flow 애플리케이션을 그래픽적으로 보기위한 flow UI 컨테이너도 필요하면 실행한다.:

```
$ docker run --rm \
       -p 3002:3000 \
       -d \
       --name flowui \
       -e API_URL=http://$DOCKER_LOCALHOST:8080 \
       -e COMPLETER_BASE_URL=http://$DOCKER_LOCALHOST:8081 \
       fnproject/flow:ui
```

## 첫번째 Fn Flow

### 첫번째 Flow 애플래케이션

Fn Java FDK를 이용하여 Maven 기반의 Java Function을 만든다. 자세한 것은 [Quickstart](./fn.md)을 읽어보기 바란다.

```
$ mkdir example-flow-function && cd example-flow-function
$ fn init --runtime=java your_dockerhub_account/flow-primes
Runtime: java
function boilerplate generated.
func.yaml created

```

### Flow 만들기

이번에 만들 애플리케이션은 입력된 번호에 해당하는 순번의 소수를 출력하는 Function 이다. 
`supply()` 를 통하여 숫자를 입력받아서 루프문을 돌면서 소수를 계산 한 후, `thenApply()` 를 통하여 결과를 저장하고, `get()`을 호출하여 HTTP를 통하여 결과를 넘겨주는 함수이다.

`src/main/java/com/example/fn/PrimeFunction.java` 을 다음의 내용으로 만든다. :

```java
package com.example.fn;

import com.fnproject.fn.api.flow.Flow;
import com.fnproject.fn.api.flow.Flows;

public class PrimeFunction {

    public String handleRequest(int nth) {

        Flow fl = Flows.currentFlow();

        return fl.supply(
                () -> {
                    int num = 1, count = 0, i = 0;

                    while (count < nth) {
                        num = num + 1;
                        for (i = 2; i <= num; i++) {
                            if (num % i == 0) {
                                break;
                            }
                        }
                        if (i == num) {
                            count = count + 1;
                        }
                    }
                    return num;
                })

                .thenApply(i -> "The " + nth + "th prime number is " + i)
                .get();
    }
}
```

`func.yaml` 을 다음과 같이 수정한다.
* The `cmd:` entry to your function's entrypoint
* The `path:` to `/primes`


```
name: your_dockerhub_account/flow-primes
version: 0.0.1
runtime: java
cmd: com.example.fn.PrimeFunction::handleRequest
path: /primes
```

### Build and Configure 

app을 생성하고 function을 deploy 한다. :

```
$ fn apps create flows-example
Successfully created app: flows-example

$ fn apps l

$ fn deploy --app flows-example 
or 
$ fn deploy --app flows-example --local
```

Local 과 Repository에 디플로이 과정은 다음과 같다.
```
~/fn/example-flow-function/shiftyou/flow-primes> fn deploy --app flows-example --local
Deploying shiftyou/primes to app: flows-example at path: /primes
Bumped to version 0.0.15
Building image shiftyou/primes:0.0.15 
Updating route /primes using image shiftyou/primes:0.0.15...
~/fn/example-flow-function/shiftyou/flow-primes> fn deploy --app flows-example        
Deploying shiftyou/primes to app: flows-example at path: /primes
Bumped to version 0.0.16
Building image shiftyou/primes:0.0.16 
Pushing shiftyou/primes:0.0.16 to docker registry...The push refers to repository [docker.io/shiftyou/primes]
0d0ae47b1692: Pushed 
e8221af68b0a: Layer already exists 
96acfe9bce4b: Layer already exists 
c18b87ea9489: Layer already exists 
7eb3fed7123d: Layer already exists 
87d7c0d076d5: Layer already exists 
aad39b3661f2: Layer already exists 
d7d312700414: Layer already exists 
0.0.16: digest: sha256:773f2924664a7f543b9670fc404aad045dae6627df8a87b9053e841d48733767 size: 1998
Updating route /primes using image shiftyou/primes:0.0.16...

```

local flow service 와 통신을 하기 위해서 설정한다. :

```
$ DOCKER_LOCALHOST=$(docker inspect --type container -f '{{.NetworkSettings.Gateway}}' fnserver)

$ fn apps config set flows-example COMPLETER_BASE_URL "http://$DOCKER_LOCALHOST:8081"
```

### Flow function 실행

`fn call`이나 curl을 통해 HTTP로 호출할 수 있다. :

```
$ echo 10 | fn call flows-example /primes
The 10th prime number is 29
```

```
$ curl -XPOST -d "10" http://localhost:8080/r/flows-example/primes
The 10th prime number is 29
```

## Asynchronous Programming Patterns

The following examples introduce the various ways in which Fn Flows enables asynchronous computation for your applications.

### Creating FlowFutures from existing values

If you already know the result of the computation:

```
  Flow fl = Flows.currentFlow();
  FlowFuture<String> stage = fl.completedValue("Hello World!);
```

or you want to create a failed stage:

```
  Flow fl = Flows.currentFlow();
  fl.failedFuture(new RuntimeException("Immediate Failure"));
```

If you want to produce a result asynchronously:

```
  Flow fl = Flows.currentFlow();
  FlowFuture<Long> stage = fl.supply(() -> {
  	 long oneHour = 60 * 60 * 1000;
  	 return System.currentTimeMillis() + oneHour;
  })
```

You can also invoke a function asynchronously and have its result complete the future once available:

```
  Flow fl = Flows.currentFlow();
  FlowFuture<HttpResponse> stage = fl.invokeFunction("myapp/myfn", HttpMethod.GET);
```

### Chaining Asynchronous Computations

By chaining FlowFutures together, you can trigger computations asynchronously once a result of a previous computation is available.

To consume the result and do some processing on it:

```
  Flow fl = Flows.currentFlow();
  FlowFuture<Long> f1 = fl.supply(() -> {
  	 long oneHour = 60 * 60 * 1000;
  	 return System.currentTimeMillis() + oneHour;
  });
  f1.thenAccept( millis -> {
  	String msg = "Time value received was " + millis;
  	System.out.println(msg);
  });
```

Similarly, you can transform the result and return the new value from the chained stage:

```
  Flow fl = Flows.currentFlow();
  FlowFuture<Long> f1 = fl.supply(() -> {
  	 long oneHour = 60 * 60 * 1000;
  	 return System.currentTimeMillis() + oneHour;
  });
  FlowFuture<Long> f2 = f1.thenApply( millis -> {
  	long seconds = millis / 1000;
  	return seconds;
  });
```

You can also chain a FlowFuture by providing a Java function that takes the previous result and itself returns a FlowFuture instance. This function is given the result of the previous computation step as its argument.

```
  Flow fl = Flows.currentFlow();
  FlowFuture<String> f1 = fl.supply(() -> "Hello");
  FlowFuture<String> f2 = f1.thenCompose( msg -> {
	return fl.supply(() -> msg + " World");
  });
```

The FlowFutures returned by _thenApply_ and _thenCompose_ are analogous to the _map_ and _flatMap_ operations provided by Java's [Stream](https://docs.oracle.com/javase/9/docs/api/java/util/stream/Stream.html) and [Optional](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html) classes. 

### Running Multiple Computations in Parallel

You can also execute two or more independent FlowFutures in parallel and combine their results once available.

To combine the results of two FlowFuture computations:

```
  Flow fl = Flows.currentFlow();
  FlowFuture<String> f1 = fl.supply(() -> "Hello");
  FlowFuture<String> f2 = fl.supply(() -> "World");
  FlowFuture<Integer> f3 = f1.thenCombine(f2, (result1, result2) -> {
  	String msg = result1 + " " + result2;
  	return msg.length();
  });

```

To wait for at least one computation to complete before invoking the next stage:

```
  Flow fl = Flows.currentFlow();
  FlowFuture<String> f1 = fl.supply(() -> {
  	try {
  		Thread.sleep((long)(Math.random() * 5000));
  	} catch(Exception e) {}
  	return "Hello";
  });
  FlowFuture<String> f2 = fl.supply(() -> {
  	try {
  		Thread.sleep((long)(Math.random() * 5000));
  	} catch(Exception e) {}
  	return "World";
  });
  fl.anyOf(f1, f2).thenApply(result -> ((String)result).toUpperCase());
```

You can also wait for all computations to complete before continuing. Simply replace the last line above with:

```
  fl.allOf(f1, f2).thenApply(ignored -> f1.get() + " " + f2.get());
```

Since the _allOf_ stage above returns a void value, you must explicitly retrieve the results of the stages you are interested in within the lambda expression.

### Handling Errors

There are several methods for handling errors with FlowFutures: 

`exceptionally` allows you to recover from the exceptional completion of a FlowFuture by transforming exceptions to the original type of the future:

```
	Flow fl = Flows.currentFlow();
	FlowFuture<Integer> f1 = fl.supply(() -> {
		if (System.currentTimeMillis() % 2L == 0L) {
			throw new RuntimeException("Error in stage");
		}
		return 100;
    }).exceptionally(err -> -1);
```

`exceptionallyCompose` is similar but allows you to handle an exception by executing one or more other nodes and attaching the subsequent FlowFuture to the result.
```
	Flow fl = Flows.currentFlow();
	FlowFuture<Integer> f1 = fl.supply(() -> {
		if (System.currentTimeMillis() % 2L == 0L) {
			throw new RuntimeException("Error in stage");
		}
		return 100;
    }).exceptionallyCompose(err -> fl.invokeFunction("./recover",new RecoveryAction());
```


`handle` is similar  to `exceptionally` but lets you deal with either the exception or the value in a single stage with a Java function that takes two parameters. In the case of success the exception value will be null and in the case of an exception the value will be null.

```
	Flow fl = Flows.currentFlow();
	FlowFuture<String> f1 = fl.supply(() -> {
		if (System.currentTimeMillis() % 2L == 0L) {
			throw new RuntimeException("Error in stage");
		}
		return 100;
	}).handle((val, err) -> {
		if (err != null){
			return "An error occurred in this function";
		} else {
			return "The result was good: " + val;
		}
	});
```



### Where Do I Go from Here?

For a more realistic application that leverages the non-blocking functionality
of Fn Flow, please take a look at the asynchronous [thumbnail generation
example](../examples/async-thumbnails/README.md).

[Advanced Topics](FnFlowsAdvancedTopics.md) provides a more in-depth treatment of data serialization and error handling with Fn Flow.