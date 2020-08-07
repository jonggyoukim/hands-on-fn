# Oracle Functions 실습

Oracle Cloud Infrastructure 의 Functions을 사용해 봅니다.


# application 생성

1. Functions(함수) 서비스를 수행하기 위하여 왼쪽 위 햄버거 아이콘을 눌러 `개발자 서비스 > 함수`를 클릭합니다.

    ![](images/fn1.png)


1. `애플리케이션 생성`을 눌러 애플리케이션을 생성합니다.

    ![](images/fn2.png)

1. 이름을 쓰고 사용하려는 VCN을 선택하고 서브넷 3개를 클릭하여 등록합니다.

    ![](images/fn3.png)

1. 아래 `생성`을 눌러 애플리케이션을 생성합니다.

    ![](images/fn4.png)

이 후의 과정은 개발과 배포 입니다.


# Context 설정

이 항목은 CloudShell을 기준으로 합니다. 로컬환경에서의 설정은 [여기](oci-functions-local.md)를 참조바랍니다.


1. CloudShell을 시작합니다.

    ![](images/fn5.png)

    CloudShell이 실행되고 프롬프트가 보일것입니다.

2. 설정된 context 의 리스트를 봅니다.

    ~~~
    $ fn list context
    ~~~

    다음과 같이 미리 생성된 context들이 보입니다.

    ![](images/fn6.png)

# Function 생성 및 배포, 실행

1. 첫번째 애플리케이션 새엇ㅇ
1. 첫번째 function 생성
1. 첫번째 function 배포
1. 첫번째 function 실행
1. 다음단계


---
### 참고
- https://www.oracle.com/webfolder/technetwork/tutorials/infographics/oci_faas_gettingstarted_quickview/functions_quickview_top/functions_quickview/index.html
