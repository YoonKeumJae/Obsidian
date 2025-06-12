---
title: "Semantic Kernel: Kernel"
---
# Semantic Kernel의 커널 이해하기 (C#)

Semantic Kernel에서 커널은 중심적인 구성 요소로, AI 애플리케이션을 실행하는 데 필요한 모든 서비스와 플러그인을 관리하는 의존성 주입(Dependency Injection) 컨테이너입니다. 모든 서비스와 플러그인을 커널에 제공하면, AI가 필요할 때 이를 원활하게 사용할 수 있습니다.

## 커널은 중심에 있습니다

커널은 네이티브 코드와 AI 서비스를 실행하는 데 필요한 모든 서비스와 플러그인을 보유하고 있으므로, Semantic Kernel SDK 내 거의 모든 구성 요소에서 에이전트를 구동하는 데 사용됩니다. 즉, Semantic Kernel에서 어떤 프롬프트나 코드를 실행하더라도, 커널은 항상 필요한 서비스와 플러그인을 검색할 수 있습니다.
![image](https://learn.microsoft.com/en-us/semantic-kernel/media/the-kernel-is-at-the-center-of-everything.png)

이러한 구조는 매우 강력합니다. 개발자는 단일 위치에서 AI 에이전트를 구성하고, 가장 중요한 모니터링을 수행할 수 있습니다. 예를 들어, 커널에서 프롬프트를 호출할 때 다음과 같은 작업이 수행됩니다:

1. 프롬프트를 실행할 최적의 AI 서비스를 선택합니다.
    
2. 제공된 프롬프트 템플릿을 사용하여 프롬프트를 생성합니다.
    
3. 프롬프트를 AI 서비스에 전송합니다.
    
4. 응답을 수신하고 구문 분석합니다.
    
5. 마지막으로 LLM의 응답을 애플리케이션에 반환합니다.
    

이 전체 과정에서 각 단계마다 이벤트와 미들웨어를 생성할 수 있습니다. 이를 통해 로깅, 사용자에게 상태 업데이트 제공, 그리고 가장 중요한 책임 있는 AI 구현과 같은 작업을 단일 위치에서 수행할 수 있습니다.

## 서비스 및 플러그인을 사용하여 커널 빌드하기

커널을 빌드하기 전에 두 가지 유형의 구성 요소를 이해해야 합니다:

|구성 요소|설명|
|---|---|
|**서비스**|AI 서비스(예: 채팅 완성)와 애플리케이션 실행에 필요한 기타 서비스(예: 로깅 및 HTTP 클라이언트)로 구성됩니다. 이는 모든 언어에서 의존성 주입을 지원하기 위해 .NET의 서비스 제공자 패턴을 모델로 삼았습니다.|
|**플러그인**|AI 서비스와 프롬프트 템플릿이 작업을 수행하는 데 사용하는 구성 요소입니다. 예를 들어, AI 서비스는 플러그인을 사용하여 데이터베이스에서 데이터를 검색하거나 외부 API를 호출하여 작업을 수행할 수 있습니다.|

커널을 생성하려면, 파일 상단에 필요한 패키지를 가져옵니다:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Plugins.Core;
```

그런 다음, 커널을 생성합니다:

```csharp
// 커널 빌더 생성
var builder = Kernel.CreateBuilder();

// Azure OpenAI 채팅 완성 서비스 추가
builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);

// 로깅 서비스 추가
builder.Services.AddLogging(c => c.AddDebug().SetMinimumLevel(LogLevel.Trace));

// 플러그인 추가
builder.Plugins.AddFromType<TimePlugin>();

// 커널 빌드
Kernel kernel = builder.Build();
```

## MCP 서버 생성하기

Semantic Kernel 인스턴스에 등록된 기능으로부터 MCP 서버를 생성하는 것을 지원합니다.

이를 위해, 일반적으로 커널을 생성한 다음, 해당 커널로부터 MCP 서버를 생성할 수 있습니다.

```csharp
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Plugins.Core;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.SemanticKernel.Mcp;

var builder = Kernel.CreateBuilder();
builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
builder.Plugins.AddFromType<TimePlugin>();
var kernel = builder.Build();

// MCP 서버 실행
var server = new McpServer(kernel);
server.Start();
```

이렇게 하면, Semantic Kernel에 등록된 기능을 활용하는 MCP 서버를 실행할 수 있습니다.

---

자세한 내용은 [Microsoft Learn의 공식 문서](https://learn.microsoft.com/en-us/semantic-kernel/concepts/kernel?pivots=programming-language-csharp)를 참조하시기 바랍니다.