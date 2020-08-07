OCI Functions 을 실습하기 위해서 몇가지 설정을 해야 합니다.

# 태넌시 설정

1. 그룹과 유저 생성
    
    기본적으로 생성이 된다.

1. 구획 생성

    필요하면 component를 설정한다.

1. VCN 과 서브넷 생성

    `네트워킹 > 가상 클라우드 네트워크` 에서 새로운 vcn을 만든다.

1. 정책 설정

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


# 개발환경 구성

Functions을 다룰 환경을 구성해보도록 합니다. 환경은 3가지의 환경을 구성할 수 있습니다. 필요한 소프트웨어를 설치하고 OCI에 배포하기 위한 환경을 구성합니다.  
가장 쉬운 방법은 아래의 Cloud Shell 환경을 사용하는 것입니다.

1. [Cloud Shell 환경](oci-functions-cloudshell.md) (추천)
1. [Local Machine 환경](oci-functions-local.md)
1. [OCI Compute Instance 환경](oci-functions-vm.md)


# Fn-Project

먼저 OCI Functions의 기본이 되는 Fn-project 를 살펴보도록 합니다.

1. [Fn Project](fn-project.md)
1. [Fn Context](fn-context.md)
1. [Oracle Functions Service](oracle-functions-service.md)

# OCI Functions

