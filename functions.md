# Oracle Functions 실습

참조 : https://www.oracle.com/webfolder/technetwork/tutorials/infographics/oci_faas_gettingstarted_quickview/functions_quickview_top/functions_quickview/index.html


# 태넌시 설정

1. 그룹과 유저 생성
    
    기본적으로 생성이 된다.

1. 구획 생성

    기본적인 구획은 자신의 Account 명으로 만들어져 있다.

1. VCN 과 서브넷 생성

    `네트워킹 > 가상 클라우드 네트워크` 에서 새로운 vcn을 만든다.

1. IAM 정책 설정

    만약 administrator 라면 다음과 같이 정책을 설정한다.
    ~~~
    Allow service FaaS to read repos in tenancy

    Allow service FaaS to use virtual-network-family in compartment <compartment-name>
    ~~~

    만약 administrator가 아니라면 다음과 같이 설정한다.
    ~~~
    Allow group <group-name> to manage repos in tenancy

    Allow group <group-name> to use virtual-network-family in compartment <compartment-name>

    Allow group <group-name> to manage functions-family in compartment <compartment-name>

    Allow group <group-name> to read metrics in compartment <compartment-name>
    ~~~



# 클라이언트 설정

1. OCI 프로파일 설정
    ~~~
    oci setup config
    ~~~
1. Docker 설치 및 시작
    윈도우10 : Docker for Windows 설치 (Windows Subsystem for Linux 를 사용하더라도 설치)
1. Fn Project CLI 설치
    
1. CLI context 생성
1. auth token 생성
1. Registry 로그인

# Function 생성 및 배포, 실행

1. 첫번째 애플리케이션 새엇ㅇ
1. 첫번째 function 생성
1. 첫번째 function 배포
1. 첫번째 function 실행
1. 다음단계
