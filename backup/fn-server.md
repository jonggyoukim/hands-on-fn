# fn-server

- fn start 로 시작됨.
- Fn server docker image를 다운로드 받아서 시작함.
- 기본포트는 8080
- Ctrl-C 로 정지
- --log-level DEBUG 옵션으로 debug 가능
- -p 8081 로 다른포트로 리슨 가능
- 기본 포트가 아닐 시 다음의 환경변수 필요
    ~~~
    export FN_API_URL=http://127.0.0.1:8081
    ~~~
- context 에서 *`api_url`* 로 정의해도 됨


