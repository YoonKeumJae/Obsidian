---
title: Power Platform
---
# About Power Platform

Power Platform: [[Power Apps]] + [[Power Automate]] + [[Power BI]] + [[Power Pages]]
파워 플랫폼은 위 4개의 주요 상품으로 구성됨. ![[Screenshot 2024-12-17 at 10.02.06.png]]
- Power Platform
	- Power Apps: 쉬운 앱 빌드
	- Power Automate: 자동화
	- Power BI: 데이터 분석
	- Power Pages: 쉬운 웹 빌드
	- Supporting tools
		- Copilot Studio
		- Power FX: low-code 언어
		- Connectors: 내/외부 앱과 연결
		- AI Builder
		- Dataverse: PowerPlatform을 위한 데이터베이스
		- Managed Environments

# Power Platform 관리 환경

![[Pasted image 20241217103755.png]]

**Microsoft Entra ID 테넌트** 아래에 각각의 **환경(Managed Environment)** 이 만들어짐. 
테넌트의 리소스를 건드릴 수 있는건 오직 그 테넌트 내부의 사용자 뿐. 
Entra ID 테넌트 아래에 만들어진 환경은 미국(Central America) 등의 지리적인 위치에 저장됨. 
환경 내부의 리소스도 마찬가지다. 
미국에 위치한 환경(Environment)에서 데이터베이스 A를 만들면 A 또한 미국의 데이터센터에 저장된다. 
데이터베이스 뿐 아니라 파워 플랫폼에서 만드는 모든 리소스들이 마찬가지다. 
환경이 위치한 지리적 위치에 모든 리소스가 함께 저장된다. 

![[Screenshot 2024-12-17 at 14.21.17.png]]

Power Platform의 관리자 센터이다. 

- 홈: 전반적인 요약
		
- **환경**: 테넌트 내의 모든 환경 나열. Microsoft Dataverse 환경 및 기타 환경(Dataverse for Teams 환경 등)이 포함됨.
		
- **분석:** Dataverse 분석, Power Automate 흐름 통계, Power Apps 세부 정보 등 Microsoft Power Platform에 대한 분석 세부 정보 제공.
		
- **청구:** 사용자 라이선스와 관련된 세부 정보가 포함.
		
- **설정:** 테넌트 수준의 설정을 검토, 관리(사용자 제어 등). 
		
- **리소스:** 테넌트에 대한 용량 통계 조회, Dynamics 365 애플리케이션과 관련된 기능을 관리, 설치.
		
- **도움말 + 지원:** 새 지원 요청 생성, 기존 요청 관리.
		
- **데이터 통합:** 미리 정의된 연결을 만들거나 추가. Microsoft Dataverse와 Salesforce 또는 SQL Server와 같은 다른 데이터 저장소 간의 연결 모니터링.
		
- **데이터:** 테넌트와 연결된 다양한 데이터 원본, 온-프레미스 데이터 게이트웨이 및 가상 네트워크 데이터 게이트웨이 관리.
		
- **정책:** 몇 가지 다양한 데이터 보안 정책 및 기타 보안 기능(고객 Lockbox, 테넌트 격리 등) 관리.
		
- **관리 센터:** Microsoft Power Platform 솔루션에 영향을 미칠 수 있는 다양한 관리 센터(Microsoft 365 관리 센터, Azure Active Directory 등)에 대한 액세스 제공.

## 데이터 손상 방지 정책: DLP 정책

환경 또는 테넌트에서 정의함. 
정보 보호, 생산성 사이 합리적인 정책을 수립하려 함. 
이를 위해 커넥터를 아래 세 가지로 나눔. 

- **비즈니스:** 비즈니스 용도 데이터를 호스트하는 커넥터.    
		
- **비즈니스 외:** 개인 용도 데이터를 호스트하는 커넥터.
		
- **차단됨:** 하나 이상의 환경에서 사용량을 제한하려는 커넥터.

기본적으로 **비즈니스 외** 로 분류됨. 
이후 Power Platform 관리자 센터에서 DLP 정책 변경 가능. 

## Dataverse

Power Platform을 위한 클라우드 기반 데이터베이스. 
클라우드 기반이기에 기본적으로 인터넷이 연결되어야 사용할 수 있다(독립적인 사용이 불가능).
아래는 Dataverse에서 제공하는 API들이다. 

![[Pasted image 20241217144401.png]]

- **Security**: 조건부 액세스 및 다단계 인증(MFA)을 허용하도록 Microsoft Entra ID 인증 처리. 행 및 열 수준까지 인증 지원, 다양한 감사 기능 제공.
		
- **logic**: 데이터 수준에서 비즈니스 논리 쉽게 적용. 사용자가 데이터와 상호 작용하는 방법에 관계없이 동일한 규칙 적용. 해당 규칙은 중복 검색, 비즈니스 규칙, 워크플로 등과 관련될 수 있음.
	    
- **Data**: 데이터를 검색 및 모델링, 데이터에 대한 유효성 검사 후 보고할 수 있도록 데이터를 셰이핑하는 컨트롤을 제공(사용 방법에 관계없이 데이터가 원하는 방식으로 표시됨).
	    
- **Storage**: Azure 클라우드에 물리적 데이터를 저장. 데이터가 상주하는 위치나 데이터 스케일링 방식에 대해 걱정하지 않아도 됨.
	    
- **Integration**: 비즈니스 요구 사항을 지원하기 위해 여러 가지 방식으로 연결됨. API, Webhook, 이벤트 및 데이터 내보내기를 통해 유연하게 데이터를 가져오고 내보낼 수 있음.

## Table 형식

Table에는 세 가지 형식이 있음. 

- **표준** - Dataverse 환경에는 여러 표준 테이블(기본 테이블)이 포함. 계정, 사업부, 연락처, 작업 및 사용자 테이블 등. 표준 테이블은 대부분 사용자 지정 가능.
    
- **관리형** - 사용자 지정할 수 없고 관리형 솔루션의 일부로 환경에 가져온 테이블.
    
- **사용자 지정** - (관리되지 않는 솔루션에서 가져온) 관리되지 않는 테이블이거나, (Dataverse 환경에서 직접 만든) 새 테이블.

이 Table에는 비즈니스 로직 또한 적용할 수 있다. 
예를 들어, 신용카드 정보에서 한도를 1000달러로 설정하면 Power Apps, Power Automate, 심지어 API를 통해서도 이 규칙이 적용된다. 
