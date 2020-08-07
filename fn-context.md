# fn context

context는 다음의 내용을 담고있다.
- docker images를 저장할 docker registry 대상
- deploy를 위한 대상.

다음의 명령으로 context 들을 열람할 수 있다. (3가지 명령은 같은 명령이다)
- fn list contexts
- fn ls context
- fn ls ctx

~~~
$ fn ls ctx
CURRENT	NAME	PROVIDER	API URL			REGISTRY
	default	default		http://localhost:8080
~~~

기본적으로 **default** 라는 이름의 context가 존재한다.
이는 로컬에서 수행하는 Fn server에게 deploy 하는 환경이다.

context를 선택하기 위해서 다음과 같이 명령하여 default context를 사용하도록 한다.
~~~
$ fn use context default
Now using context: default
~~~

다시 리스트를 보면 CURRENT 항목에 * 마크가 되어있음을 알 수 있다.
~~~
$ fn ls ctx
CURRENT	NAME	PROVIDER	API URL			REGISTRY
*	default	default		http://localhost:8080
~~~

로컬에서 사용하니 임의로 registry를 등록해 보도록 한다.
~~~
$ fn update context registry fndemouser
Current context updated registry with fndemouser
~~~

다시 리스트를 보면 REGISTRY 항목에 fndemouser 로 변경됨을 알 수 있다.
~~~
$ fn ls ctx
CURRENT	NAME	PROVIDER	API URL			REGISTRY
*	default	default		http://localhost:8080	fndemouser
~~~

만약 docker hub를 사용하다면 *fndemouser* 항목을 Docker Hub username 으로 변경한다.
~~~
$ fn update context registry your-docker-hub-user-name
~~~


# Context 새로 생성하기
새로 context 를 생성하기 위해서는 `fn create context`를 사용한다.
~~~
$ fn create context my-local-ctx --api-url http://localhost:8080 --registry fnlocal
Successfully created context: my-local-ctx
~~~
- --provider : repository 프로바이더를 지정한다. 원하는 명을 사용한다. 지정하지 않으면 *default* 가 사용된다.
- --api-url : Fn server 의 주소와 포트를 지정한다.로컬서버가 될 수있고 리모트 서버가 될 수도 있다.
- --registry : registry의 이름이다. 로컬 개발을 위해서는 지정한 이름 아무거나 사용되고 Docker hub에서 사용하려면 Docker hub의 username 이어야 한다.

context의 리스트를 보면 다음과 같이 나타난다.
~~~
$ fn list ctx
CURRENT	NAME		PROVIDER	API URL			REGISTRY
*	default		default		http://localhost:8080	fndemouser
	my-local-ctx	default		http://localhost:8080	fnlocal
~~~

context의 정보는 `~/.fn/contexts`에 저장이 된다.
~~~
$ ls ~/.fn/contexts
default.yaml      my-local-ctx.yaml
~~~

default.yaml 파일을 보면 다음과 같이 구성되어 있다.
~~~
$ cat ~/.fn/contexts/default.yaml
api-url: http://localhost:8080
provider: default
registry: fndemouser
~~~

현재 context를 바꾸려면 다음과 같이 바꾼다.
~~~
$ fn use context my-local-ctx
Now using context: my-local-ctx
~~~

다음과 같이 CURRENT 가 바뀌었음을 알 수 있다.
~~~
$ fn list ctx
CURRENT	NAME		PROVIDER	API URL			REGISTRY
	default		default		http://localhost:8080	fndemouser
*	my-local-ctx	default		http://localhost:8080	fnlocal
~~~

