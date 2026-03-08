# Architecture Diagrams

This document contains detailed architecture diagrams for the Universal Agent Protocol proposal.

## System Overview

```mermaid
graph TB
    User[👤 User] --> App[📱 Application]
    App --> ACP[ACP Client]
    ACP <--> Agent[🤖 Agent]
    App --> MCP[MCP Server]
    MCP -.expose tools.-> ACP
    Agent --> LLM[🧠 LLM/Reasoning]
    
    style App fill:#e1f5ff
    style Agent fill:#fff4e1
    style MCP fill:#f0f0f0
```

## Protocol Stack

```mermaid
graph LR
    subgraph "Application Side"
        App[Application] --> ACPC[ACP Client]
        App --> MCPS[MCP Server]
        MCPS --> Tools[Domain Tools]
    end
    
    subgraph "Agent Side"
        ACPS[ACP Server] --> Reasoning[AI Reasoning]
        ACPS --> ToolInvoke[Tool Invocation]
    end
    
    ACPC <-->|"ACP Protocol<br/>(Extended)"| ACPS
    
    style App fill:#e3f2fd
    style ACPS fill:#fff3e0
    style Tools fill:#f1f8e9
```

## Communication Flow

```mermaid
sequenceDiagram
    participant User
    participant App as Application<br/>(ACP Client)
    participant MCP as MCP Server
    participant Agent as Agent<br/>(ACP Server)
    participant LLM as LLM
    
    User->>App: Prompt: "Create a rectangle"
    App->>Agent: send_prompt
    
    Agent->>App: get_capabilities
    App->>MCP: list_tools
    MCP-->>App: [tools]
    App-->>Agent: client_capabilities {tools}
    
    Agent->>LLM: plan with available tools
    LLM-->>Agent: [invoke create_rectangle]
    
    Agent->>App: invoke_tool(create_rectangle, params)
    App->>MCP: execute_tool
    MCP-->>App: result
    App-->>Agent: tool_result(success)
    
    Agent->>User: Response: "Rectangle created!"
```

## Tool Discovery Flow

```mermaid
flowchart TD
    Start([Connection Established]) --> Handshake[ACP Handshake]
    Handshake --> CapReq{Agent Requests<br/>Capabilities?}
    
    CapReq -->|Yes| ListTools[Client Lists<br/>MCP Tools]
    ListTools --> Format[Format as<br/>ACP Capabilities]
    Format --> Send[Send to Agent]
    Send --> Store[Agent Stores<br/>Available Tools]
    
    CapReq -->|No| Wait[Wait for<br/>First Prompt]
    Wait --> Discovery[On-Demand<br/>Discovery]
    Discovery --> ListTools
    
    Store --> Ready([Ready for<br/>Tool Invocation])
    
    style Start fill:#c8e6c9
    style Ready fill:#c8e6c9
    style ListTools fill:#fff9c4
    style Store fill:#f8bbd0
```

## Tool Invocation Flow

```mermaid
flowchart TD
    Prompt([User Prompt]) --> AgentReason[Agent Reasoning]
    AgentReason --> NeedTool{Need Tool?}
    
    NeedTool -->|No| Direct[Direct Response]
    Direct --> Done([Done])
    
    NeedTool -->|Yes| SelectTool[Select Tool<br/>from Available]
    SelectTool --> BuildReq[Build Tool<br/>Request]
    BuildReq --> SendReq[Send invoke_tool]
    SendReq --> Client[ACP Client<br/>Receives]
    Client --> ValidateTool[Validate Tool<br/>Exists]
    
    ValidateTool -->|Not Found| ErrNotFound[Error:<br/>Tool Not Found]
    ErrNotFound --> AgentHandle[Agent Handles<br/>Error]
    
    ValidateTool -->|Found| ValidateParams[Validate<br/>Parameters]
    ValidateParams -->|Invalid| ErrParams[Error:<br/>Invalid Params]
    ErrParams --> AgentHandle
    
    ValidateParams -->|Valid| CheckSecurity[Security<br/>Check]
    CheckSecurity -->|Denied| ErrDenied[Error:<br/>Permission Denied]
    ErrDenied --> AgentHandle
    
    CheckSecurity -->|Approved| Execute[Execute via<br/>MCP Server]
    Execute --> Success{Execution<br/>Success?}
    
    Success -->|Yes| ReturnResult[Return<br/>tool_result]
    Success -->|No| ReturnError[Return Error]
    
    ReturnResult --> AgentProcess[Agent Processes<br/>Result]
    ReturnError --> AgentHandle
    
    AgentProcess --> More{Need More<br/>Tools?}
    More -->|Yes| SelectTool
    More -->|No| FinalResp[Generate Final<br/>Response]
    
    AgentHandle --> Retry{Retryable?}
    Retry -->|Yes| SelectTool
    Retry -->|No| FinalResp
    
    FinalResp --> Done
    
    style Prompt fill:#e1f5fe
    style Done fill:#c8e6c9
    style Execute fill:#fff9c4
    style Success fill:#ffe0b2
```

## Integration Patterns

### Pattern 1: Embedded MCP Server

```mermaid
graph TB
    subgraph "Single Application Process"
        App[Application<br/>Core Logic]
        ACPC[ACP Client<br/>Implementation]
        MCPS[Embedded<br/>MCP Server]
        Tools[Domain-Specific<br/>Tools]
        
        App --> ACPC
        App --> MCPS
        MCPS --> Tools
        MCPS -.exposes.-> ACPC
    end
    
    ACPC <-->|ACP Protocol| Agent[External<br/>Agent]
    
    style App fill:#e3f2fd
    style MCPS fill:#fff3e0
    style Agent fill:#f3e5f5
```

### Pattern 2: External MCP Servers

```mermaid
graph TB
    subgraph "Application"
        App[Application<br/>Core Logic]
        ACPC[ACP Client<br/>Implementation]
        Bridge[MCP Client<br/>Bridge]
    end
    
    subgraph "MCP Ecosystem"
        FS[Filesystem<br/>MCP Server]
        Git[GitHub<br/>MCP Server]
        DB[Database<br/>MCP Server]
    end
    
    App --> ACPC
    App --> Bridge
    Bridge --> FS
    Bridge --> Git
    Bridge --> DB
    
    FS -.exposes.-> Bridge
    Git -.exposes.-> Bridge
    DB -.exposes.-> Bridge
    Bridge -.aggregates.-> ACPC
    
    ACPC <-->|ACP Protocol| Agent[External<br/>Agent]
    
    style App fill:#e3f2fd
    style FS fill:#e8f5e9
    style Git fill:#e8f5e9
    style DB fill:#e8f5e9
    style Agent fill:#f3e5f5
```

### Pattern 3: Hybrid

```mermaid
graph TB
    subgraph "Application"
        App[Application<br/>Core Logic]
        ACPC[ACP Client]
        MCPS[Embedded<br/>Domain Tools]
        Bridge[External<br/>MCP Bridge]
    end
    
    subgraph "External Services"
        FS[Filesystem]
        Cloud[Cloud APIs]
    end
    
    App --> ACPC
    App --> MCPS
    App --> Bridge
    
    Bridge --> FS
    Bridge --> Cloud
    
    MCPS -.domain tools.-> ACPC
    Bridge -.general tools.-> ACPC
    
    ACPC <-->|ACP Protocol| Agent[Agent]
    
    style App fill:#e3f2fd
    style MCPS fill:#fff3e0
    style FS fill:#e8f5e9
    style Cloud fill:#e8f5e9
    style Agent fill:#f3e5f5
```

## Comparison: ACP vs ACP+MCP vs AG-UI

```mermaid
graph TB
    subgraph "Protocols Comparison"
        direction LR
        
        subgraph "ACP (Current)"
            ACP1[Stdio Transport]
            ACP2[Session Mgmt]
            ACP3[File Operations]
            ACP4[Code Specific]
        end
        
        subgraph "ACP + MCP (Proposed)"
            ACPM1[Stdio/WebSocket]
            ACPM2[Session Mgmt]
            ACPM3[Tool Discovery]
            ACPM4[Any Domain]
            ACPM5[MCP Integration]
        end
        
        subgraph "AG-UI"
            AGUI1[Event-based]
            AGUI2[Agent State]
            AGUI3[UI Rendering]
            AGUI4[User-facing Apps]
        end
    end
    
    style ACP1 fill:#bbdefb
    style ACPM1 fill:#c5e1a5
    style AGUI1 fill:#ffccbc
```

## Complete System Architecture

```mermaid
graph TB
    subgraph "User Interaction Layer"
        User[👤 User]
        UI[Application UI]
    end
    
    subgraph "Presentation Layer (Optional)"
        AGUI[AG-UI or Similar]
        Canvas[UI Components]
        Forms[Forms & Dialogs]
    end
    
    subgraph "Logic Layer"
        ACPC[ACP Client]
        ToolReg[Tool Registry]
        Security[Security Layer]
        
        subgraph "MCP Integration"
            Embedded[Embedded Tools]
            External[External Tools]
            Custom[Custom Tools]
        end
    end
    
    subgraph "Application Core"
        AppLogic[Business Logic]
        Data[Data Layer]
        Native[Native APIs]
    end
    
    subgraph "Agent Side"
        ACPS[ACP Server]
        Planning[Planning & Reasoning]
        LLM[LLM]
        Memory[Context & Memory]
    end
    
    User --> UI
    UI --> ACPC
    
    ACPC --> AGUI
    AGUI --> Canvas
    AGUI --> Forms
    
    ACPC --> ToolReg
    ToolReg --> Security
    Security --> Embedded
    Security --> External
    Security --> Custom
    
    Embedded --> AppLogic
    Custom --> AppLogic
    AppLogic --> Data
    AppLogic --> Native
    
    ACPC <-->|ACP Protocol<br/>+ Tool Invocation| ACPS
    
    ACPS --> Planning
    Planning --> LLM
    Planning --> Memory
    
    style User fill:#e1f5fe
    style ACPC fill:#fff3e0
    style ACPS fill:#f3e5f5
    style AGUI fill:#ede7f6
    style LLM fill:#fff9c4
```

## Data Flow: End-to-End

```mermaid
sequenceDiagram
    autonumber
    
    participant U as User
    participant UI as App UI
    participant ACP as ACP Client
    participant MCP as MCP Tools
    participant Net as Network
    participant Agent as Agent
    participant LLM as LLM
    participant A2UI as A2UI Layer
    
    U->>UI: User input/action
    UI->>ACP: Forward prompt
    
    rect rgb(230, 240, 255)
        Note over ACP,Agent: Initial Handshake & Discovery
        ACP->>Agent: Connect + capabilities
        Agent->>ACP: Request tool list
        ACP->>MCP: List available tools
        MCP-->>ACP: Tool definitions
        ACP-->>Agent: Tool capabilities
    end
    
    rect rgb(255, 245, 230)
        Note over Agent,LLM: Planning Phase
        Agent->>LLM: Plan with tools
        LLM-->>Agent: Execution plan
    end
    
    rect rgb(240, 255, 240)
        Note over Agent,MCP: Execution Phase
        loop For each tool call
            Agent->>ACP: invoke_tool(name, args)
            ACP->>MCP: Execute tool
            MCP-->>ACP: Result
            ACP-->>Agent: tool_result
        end
    end
    
    rect rgb(255, 240, 245)
        Note over Agent,A2UI: Optional: Show Progress
        Agent->>A2UI: Render progress UI
        A2UI-->>UI: Display progress
    end
    
    rect rgb(245, 245, 255)
        Note over Agent,U: Response Phase
        Agent->>LLM: Generate response
        LLM-->>Agent: Final response
        Agent->>ACP: Send response
        ACP->>UI: Display result
        UI->>U: Show result
    end
```

## Security & Approval Flow

```mermaid
flowchart TD
    ToolReq([Tool Invocation<br/>Request]) --> Parse[Parse Request]
    Parse --> CheckTool{Tool<br/>Exists?}
    
    CheckTool -->|No| Err404[Error 404:<br/>Tool Not Found]
    Err404 --> Return([Return Error])
    
    CheckTool -->|Yes| CheckParams{Valid<br/>Parameters?}
    CheckParams -->|No| Err400[Error 400:<br/>Invalid Parameters]
    Err400 --> Return
    
    CheckParams -->|Yes| CheckSensitive{Sensitive<br/>Operation?}
    
    CheckSensitive -->|No| Execute[Execute Tool]
    
    CheckSensitive -->|Yes| CheckPolicy{Has Auto<br/>Approval?}
    CheckPolicy -->|Yes| Execute
    
    CheckPolicy -->|No| UserPrompt[Prompt User<br/>for Approval]
    UserPrompt --> UserDecision{User<br/>Approves?}
    
    UserDecision -->|No| Err403[Error 403:<br/>Permission Denied]
    Err403 --> Return
    
    UserDecision -->|Yes| Remember{Remember<br/>Choice?}
    Remember -->|Yes| SavePolicy[Save to<br/>Policy Store]
    Remember -->|No| Execute
    SavePolicy --> Execute
    
    Execute --> Success{Success?}
    Success -->|Yes| ReturnResult[Return Result]
    Success -->|No| ReturnError[Return Error]
    
    ReturnResult --> Done([Done])
    ReturnError --> Done
    
    style ToolReq fill:#e1f5fe
    style Done fill:#c8e6c9
    style Execute fill:#fff9c4
    style UserPrompt fill:#ffccbc
    style Err403 fill:#ffcdd2
    style Err404 fill:#ffcdd2
    style Err400 fill:#ffcdd2
```

---

## ASCII Diagrams (for Markdown compatibility)

### System Overview (ASCII)

```
┌─────────────────────────────────────────────────┐
│  APPLICATION (Client)                           │
│                                                  │
│  ┌───────────────────────────────────────────┐ │
│  │ ACP Client Implementation                  │ │
│  │  • Bidirectional communication            │ │
│  │  • Session management                     │ │
│  │  • Tool capability advertising            │ │
│  └───────────────┬───────────────────────────┘ │
│                  │                               │
│  ┌───────────────▼───────────────────────────┐ │
│  │ MCP Server(s)                              │ │
│  │  • Tool definitions                       │ │
│  │  • Tool execution                         │ │
│  │  • Domain-specific logic                  │ │
│  └───────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
                   ↕
          ┌────────┴────────┐
          │   ACP Protocol   │
          │   (Extended)     │
          │                  │
          │ • Prompts        │
          │ • Responses      │
          │ • Tool discovery │
          │ • Tool invocation│
          └────────┬─────────┘
                   ↕
┌─────────────────────────────────────────────────┐
│  AGENT (Server)                                  │
│                                                  │
│  ┌───────────────────────────────────────────┐ │
│  │ ACP Server Implementation                  │ │
│  │  • Prompt handling                        │ │
│  │  • Tool discovery                         │ │
│  │  • Tool invocation                        │ │
│  └───────────────┬───────────────────────────┘ │
│                  │                               │
│  ┌───────────────▼───────────────────────────┐ │
│  │ AI Reasoning Engine                        │ │
│  │  • LLM integration                        │ │
│  │  • Planning & orchestration               │ │
│  │  • Response generation                    │ │
│  └───────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

### Communication Flow (ASCII)

```
User                App (ACP)        MCP Server       Agent           LLM
 │                     │                  │              │              │
 ├─ Prompt ───────────>│                  │              │              │
 │                     ├─ Forward ───────>│              │              │
 │                     │<─ Tools ─────────┤              │              │
 │                     ├─ Capabilities ──────────────────>│              │
 │                     │                  │              ├─ Plan ───────>│
 │                     │                  │              │<─ Actions ────┤
 │                     │<─ invoke_tool ──────────────────┤              │
 │                     ├─ Execute ───────>│              │              │
 │                     │<─ Result ────────┤              │              │
 │                     ├─ tool_result ───────────────────>│              │
 │                     │                  │              ├─ Finalize ───>│
 │                     │                  │              │<─ Response ───┤
 │<─ Response ─────────┼──────────────────────────────────┤              │
 │                     │                  │              │              │
```

### Protocol Stack (ASCII)

```
┌────────────────────────────────────────────────────┐
│                                                     │
│         PRESENTATION LAYER (Optional)              │
│         ┌──────────────────────────┐              │
│         │    A2UI Protocol         │              │
│         │  • Declarative UI        │              │
│         │  • Component rendering   │              │
│         │  • Visual feedback       │              │
│         └──────────────────────────┘              │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│         LOGIC LAYER                                │
│         ┌──────────────────────────┐              │
│         │  ACP + MCP Integration   │              │
│         │  • Tool discovery        │              │
│         │  • Tool invocation       │              │
│         │  • Context passing       │              │
│         └──────────────────────────┘              │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│         APPLICATION LAYER                          │
│         ┌──────────────────────────┐              │
│         │  Domain-Specific Logic   │              │
│         │  • Business rules        │              │
│         │  • Data access           │              │
│         │  • Native capabilities   │              │
│         └──────────────────────────┘              │
│                                                     │
└────────────────────────────────────────────────────┘
```

---

**Note:** For best viewing of Mermaid diagrams, use GitHub's native markdown renderer or any Mermaid-compatible viewer.
