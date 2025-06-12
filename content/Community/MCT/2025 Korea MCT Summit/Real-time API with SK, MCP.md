---
title: Real-time API with SK, MCP
---
Semantic Kernel의 Python SDK에서는 **OpenAI Realtime API**와 **Model Context Protocol(MCP)**을 모두 지원하여, 실시간 스트리밍 AI 워크플로우를 유연하게 구성할 수 있습니다. 구체적으로, Realtime API는 WebSocket 및 WebRTC 기반의 **Realtime Client**를 통해 텍스트·오디오 스트리밍과 함수 호출을 제공하고, MCP는 SK를 호스트나 서버로 동작시켜 외부 도구와 에이전트를 표준 프로토콜로 연동할 수 있습니다. 이를 결합하면, SK 내부에서 실시간 입출력 기능을 Kernel 함수로 래핑(wrap)하고, MCP 플러그인 혹은 서버로 노출하여 다른 MCP 호스트/클라이언트와 상호운용하는 아키텍처를 구현할 수 있습니다.

## 1. OpenAI Realtime API 지원

### 1.1 지원 현황

- SK Python 패키지(`semantic-kernel[realtime]`) 설치만으로 실험적 Realtime API 통합이 활성화됩니다 ([Realtime AI Integrations for Semantic Kernel | Microsoft Learn](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/realtime)).
    
- 현재 **Python 전용**, **Experimental** 단계이며, .NET SDK에서는 아직 공식 지원되지 않습니다 ([Realtime AI Integrations for Semantic Kernel | Microsoft Learn](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/realtime)).
    

### 1.2 제공 기능

- **WebSocket** 기반 클라이언트(`OpenAIRealtimeWebsocket`)와
    
- **WebRTC** 기반 클라이언트(`OpenAIRealtimeWebRTC`)가 제공되어
    
    - 텍스트·오디오 양쪽을 스트리밍하고
        
    - 함수 호출(Function Calling)이 가능합니다 ([Realtime AI Integrations for Semantic Kernel | Microsoft Learn](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/realtime)).
        

### 1.3 사용 예시

```python
from semantic_kernel.connectors.ai.open_ai import OpenAIRealtimeWebsocket, OpenAIRealtimeExecutionSettings

# 1) Realtime 클라이언트 생성
realtime_client = OpenAIRealtimeWebsocket()

# 2) 실행 설정: 음성 지정 등
settings = OpenAIRealtimeExecutionSettings(voice="alloy")

# 3) 세션 관리 및 메시지 송수신
async with realtime_client(settings=settings, create_response=True):
    async for event in realtime_client.receive():
        if event.type == "text":
            print(event.text.text, end="")
```

이처럼 `receive()`가 **async generator**로 동작하여, 도착하는 텍스트나 오디오 청크를 실시간으로 처리할 수 있습니다 ([Realtime AI Integrations for Semantic Kernel | Microsoft Learn](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/realtime)).

#### 참고

- 자세한 구현 코드는 [GitHub 리포지토리](https://github.com/microsoft/semantic-kernel/blob/main/python/semantic_kernel/connectors/ai/open_ai/services/open_ai_realtime.py)에서 확인 가능합니다 ([semantic-kernel/python/semantic_kernel/connectors/ai/open_ai ...](https://github.com/microsoft/semantic-kernel/blob/main/python/semantic_kernel/connectors/ai/open_ai/services/open_ai_realtime.py?utm_source=chatgpt.com)).
    
- 고급 예제로 **Azure Communication Services(ACS)**를 이용해 전화 통화 시나리오를 다룬 데모도 제공됩니다 ([Realtime AI Integrations for Semantic Kernel | Microsoft Learn](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/realtime)).
    

## 2. Model Context Protocol(MCP) 지원

### 2.1 MCP 개요

- MCP는 Anthropic이 제안한 **모델·도구·에이전트 상호 운용 표준 프로토콜**로, stdio·SSE·WebSocket 등 다양한 전송 방식을 지원합니다 ([Semantic Kernel adds Model Context Protocol (MCP) support for Python | Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-adds-model-context-protocol-mcp-support-for-python/)).
    

### 2.2 SK의 MCP 기능

- **MCP 호스트(Client)**: SK가 외부 MCP 서버를 플러그인처럼 연결하여 도구와 프롬프트를 호출할 수 있습니다
    
- **MCP 서버(Server)**: SK의 함수·프롬프트를 MCP 서버로 노출하여, 다른 호스트에서 표준 방식으로 호출할 수 있습니다 ([Semantic Kernel adds Model Context Protocol (MCP) support for Python | Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-adds-model-context-protocol-mcp-support-for-python/)).
    

#### 2.2.1 예시: MCP 호스트

```python
from semantic_kernel.connectors.mcp import MCPStdioPlugin

async with MCPStdioPlugin(
    name="ReleaseNotes",
    command="uv",
    args=["--directory=python/samples/demos/mcp_server","run","mcp_server_with_sampling.py"],
) as plugin:
    # plugin을 Kernel 플러그인으로 사용
    ...
```

이 구성으로 로컬 stdio MCP 서버를 SK 에이전트에서 바로 호출할 수 있습니다 ([Semantic Kernel adds Model Context Protocol (MCP) support for Python | Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-adds-model-context-protocol-mcp-support-for-python/)).

#### 2.2.2 예시: MCP 서버

```python
from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import OpenAIChatCompletion

kernel = Kernel()
kernel.add_service(OpenAIChatCompletion(service_id="default"))

# SK를 MCP 서버로 실행
server = kernel.as_mcp_server(server_name="sk")
import anyio
from mcp.server.stdio import stdio_server

async def handle_stdin():
    async with stdio_server() as (read, write):
        await server.run(read, write, server.create_initialization_options())

anyio.run(handle_stdin)
```

이제 `sk`라는 MCP 서버로 SK 함수와 프롬프트가 노출됩니다 ([Semantic Kernel adds Model Context Protocol (MCP) support for Python | Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-adds-model-context-protocol-mcp-support-for-python/)).

## 3. Realtime API와 MCP 통합 시나리오

### 3.1 아키텍처 개요

1. **Realtime Client**를 Kernel에 등록하여
    
2. **KernelFunction**으로 래핑(wrap)
    
3. Wrap된 함수 또는 에이전트를 **MCP 서버**로 노출
    
4. 다른 MCP 호스트/클라이언트가 표준 프로토콜로 호출
    

이렇게 하면 **실시간 스트리밍 기능**을 MCP 기반 툴 체인에 자연스럽게 통합할 수 있습니다 ([Realtime AI Integrations for Semantic Kernel | Microsoft Learn](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/realtime), [Semantic Kernel adds Model Context Protocol (MCP) support for Python | Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-adds-model-context-protocol-mcp-support-for-python/)).

### 3.2 주의사항 및 제한

- 두 기능 모두 **Python SDK 전용**이며, 현재 .NET SDK로는 Realtime API 통합이 제공되지 않습니다 ([Realtime AI Integrations for Semantic Kernel | Microsoft Learn](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/realtime)).
    
- MCP C# SDK를 활용하면 .NET 앱에서 MCP 서버/호스트로 연동할 수 있으나, Realtime API는 Python에서만 사용 가능합니다 ([Integrating Model Context Protocol Tools with Semantic Kernel: A Step-by-Step Guide | Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/integrating-model-context-protocol-tools-with-semantic-kernel-a-step-by-step-guide/)).
    

## 4. 결론

Semantic Kernel Python SDK의 **Realtime API**와 **MCP** 기능을 결합하면, 텍스트·오디오 실시간 스트리밍과 에이전트 간 표준화된 상호운용성을 한 번에 달성할 수 있습니다. 이를 통해 복잡한 멀티모달 에이전트를 구축하거나, 여러 프로세스와 네트워크 경계를 넘나드는 파이프라인을 손쉽게 설계할 수 있습니다. 앞으로 .NET 쪽 Realtime 기능이 추가되면, Python에서 시도한 아키텍처를 .NET 환경으로도 확장해볼 수 있을 것입니다.