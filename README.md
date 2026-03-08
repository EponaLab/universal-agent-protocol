# Universal Agent Protocol: Making ACP Universal Through MCP Integration

**A proposal for transforming the Agent Client Protocol (ACP) into a truly universal copilot/agent protocol by integrating Model Context Protocol (MCP) for bidirectional tool invocation.**

---

## 📋 Table of Contents

1. [The Problem](#the-problem)
2. [The Opportunity](#the-opportunity)
3. [What Exists Today](#what-exists-today)
4. [The Proposal](#the-proposal)
5. [Architecture](#architecture)
6. [Use Cases](#use-cases)
7. [Implementation Guide](#implementation-guide)
8. [Comparison with A2UI](#comparison-with-a2ui)
9. [Future Vision](#future-vision)
10. [References](#references)

---

## 🔴 The Problem

### Current Landscape

Today's AI agent-application integration landscape is fragmented:

**Domain-Specific Protocols:**
- Each domain needs custom protocols (code editors, design tools, data tools)
- No standard for exposing application capabilities to agents
- Proprietary solutions (GitHub Copilot, Cursor, etc.) lock users into specific ecosystems
- Developers must build custom integrations for each application type

**Limitations of Existing Protocols:**

**ACP (Agent Client Protocol):**
- ✅ Excellent for code editor ↔ coding agent communication
- ❌ Code-editor-specific commands (file operations, terminal access)
- ❌ Not designed for other domains (design, data, creative tools)
- ❌ Limited extensibility without protocol changes

**MCP (Model Context Protocol):**
- ✅ Great for exposing tools/capabilities to agents
- ✅ Rich ecosystem of servers
- ❌ No standardized client-side integration protocol
- ❌ Typically agent → server (not bidirectional app ↔ agent)

**The Gap:**
There's no universal protocol that allows **any application** to become an AI copilot client by exposing its domain-specific capabilities in a standardized way.

### Why This Matters

**For Application Developers:**
- Must build custom agent integrations from scratch
- No shared tooling or best practices
- Each integration is a separate engineering effort

**For Agent Developers:**
- Must support multiple proprietary protocols
- Can't leverage capabilities across applications
- Integration friction limits adoption

**For Users:**
- Locked into specific tool combinations
- Can't use their preferred agent across applications
- Fragmented experience across different tools

---

## 🎯 The Opportunity

### Vision: Universal Agent-Application Protocol

Imagine a world where **any application** can become an AI copilot client by:

1. **Implementing ACP** for bidirectional communication
2. **Exposing capabilities via MCP** for agent tool invocation
3. **Using standardized patterns** for discovery and orchestration

### What This Enables

**For Design Tools (Figma, Sketch, Canva):**
```
Agent: "Create a login form with email and password fields"
→ Invokes: create_text_field(), create_button(), apply_style()
```

**For Data Tools (Excel, Tableau, Postgres):**
```
Agent: "Find anomalies in column C and create a visualization"
→ Invokes: execute_query(), analyze_data(), create_chart()
```

**For Creative Tools (Photoshop, Premiere, DAW):**
```
Agent: "Remove background and enhance colors"
→ Invokes: select_subject(), remove_background(), adjust_levels()
```

**For Enterprise Apps (CRM, ERP):**
```
Agent: "Draft follow-up emails for opportunities closing this week"
→ Invokes: query_opportunities(), generate_email(), schedule_send()
```

### Key Benefits

1. **Universality:** One protocol works across all domains
2. **Extensibility:** Add new capabilities without protocol changes
3. **Ecosystem Leverage:** Reuse existing MCP servers and tooling
4. **Standards-Based:** Built on open protocols (ACP + MCP)
5. **Composability:** Mix capabilities from multiple sources
6. **Developer-Friendly:** Clear patterns and tooling

---

## 📚 What Exists Today

### Agent Client Protocol (ACP)

**Status:** ✅ **Production-Ready**

**What it is:**
- Standardized communication protocol for code editors ↔ AI coding agents
- Maintained by Zed Industries
- Growing ecosystem (Zed, Google Gemini CLI, OpenClaw)

**Core Features:**
- Stdio-based transport
- Session management
- File context passing
- Code edit operations
- Request/response patterns

**Resources:**
- Website: https://agentclientprotocol.com
- TypeScript SDK: https://github.com/agentclientprotocol/typescript-sdk
- Documentation: https://agentclientprotocol.com/protocol/overview

**Current Use Cases:**
- ✅ Zed editor integration
- ✅ IDE copilots
- ✅ Code generation workflows
- ✅ Terminal/build integration

**Limitations:**
- Domain-specific to code editing
- Fixed command set (read_file, write_file, etc.)
- Not designed for non-code domains

### Model Context Protocol (MCP)

**Status:** ✅ **Production-Ready**

**What it is:**
- Protocol for exposing tools and context to AI models
- Maintained by Anthropic
- Growing ecosystem of servers

**Core Features:**
- Tool definition format
- Capability discovery
- Structured tool calling
- Resource management
- Security boundaries

**Resources:**
- Website: https://modelcontextprotocol.io
- Specification: https://spec.modelcontextprotocol.io
- Servers: https://github.com/modelcontextprotocol/servers

**Existing MCP Servers:**
- Filesystem operations
- GitHub API
- Database queries (Postgres, SQLite)
- Web search
- Slack integration
- Google Drive
- Many more...

**Current Use Cases:**
- ✅ Claude Desktop integration
- ✅ Tool calling for LLMs
- ✅ Context management
- ✅ Resource access

**Limitations:**
- Typically agent → server (one-way)
- No standard for client-side application integration
- Not designed for interactive application control

### A2UI (Agent-to-UI)

**Status:** ✅ **Implemented in OpenClaw**

**What it is:**
- Declarative UI rendering protocol
- Agents send UI structures to render on devices
- One-way: Agent → UI

**Core Features:**
- Component-based (Column, Row, Text, Button, etc.)
- JSONL format
- WebView/Canvas rendering
- Mobile-friendly (iOS/Android)

**Use Cases:**
- ✅ Agent-generated interfaces
- ✅ Progress indicators
- ✅ Form collection
- ✅ Results display

**Scope:**
- **Different layer** than ACP/MCP
- Focuses on visual output, not capability invocation
- Complementary to logic layer protocols

---

## 💡 The Proposal

### Core Concept: ACP + MCP = Universal Protocol

**Extend ACP with bidirectional MCP tool invocation:**

1. **Clients expose tools via MCP format** through ACP
2. **Agents discover available tools** from clients
3. **Agents invoke client tools** as needed
4. **Applications become domain-agnostic** via tool abstraction

### Architecture Overview

```
┌─────────────────────────────────────────────────┐
│  APPLICATION (Client)                           │
│                                                  │
│  ┌───────────────────────────────────────────┐ │
│  │ ACP Client Implementation                  │ │
│  │  - Sends prompts to agent                 │ │
│  │  - Receives responses                     │ │
│  │  - EXPOSES TOOLS via MCP format           │ │
│  └───────────────┬───────────────────────────┘ │
│                  │                               │
│  ┌───────────────▼───────────────────────────┐ │
│  │ MCP Tool Server(s)                         │ │
│  │  - Domain-specific capabilities           │ │
│  │  - file_read(), execute_query()           │ │
│  │  - create_shape(), apply_filter()         │ │
│  │  - Any application-specific tools         │ │
│  └───────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
                   ↕
          ACP Protocol (Extended)
          - Prompts & responses
          - Tool discovery
          - Tool invocation
                   ↕
┌─────────────────────────────────────────────────┐
│  AGENT (Server)                                  │
│  - Receives prompts                              │
│  - Discovers client tools (MCP format)          │
│  - Invokes tools as needed                      │
│  - Returns results & responses                   │
└─────────────────────────────────────────────────┘
```

### Protocol Extensions

#### 1. Tool Discovery Phase

**Client → Agent: Advertise Capabilities**

```json
{
  "type": "client_capabilities",
  "capabilities": {
    "tools": [
      {
        "name": "read_file",
        "description": "Read file contents from the workspace",
        "inputSchema": {
          "type": "object",
          "properties": {
            "path": {
              "type": "string",
              "description": "Relative path to file"
            }
          },
          "required": ["path"]
        }
      },
      {
        "name": "create_shape",
        "description": "Create a shape in the design canvas",
        "inputSchema": {
          "type": "object",
          "properties": {
            "type": {
              "type": "string",
              "enum": ["rectangle", "circle", "polygon"]
            },
            "x": {"type": "number"},
            "y": {"type": "number"},
            "width": {"type": "number"},
            "height": {"type": "number"}
          },
          "required": ["type", "x", "y"]
        }
      }
    ]
  }
}
```

#### 2. Tool Invocation

**Agent → Client: Invoke Tool**

```json
{
  "type": "invoke_tool",
  "requestId": "req_123",
  "tool": "create_shape",
  "arguments": {
    "type": "rectangle",
    "x": 100,
    "y": 100,
    "width": 200,
    "height": 150
  }
}
```

**Client → Agent: Tool Result**

```json
{
  "type": "tool_result",
  "requestId": "req_123",
  "result": {
    "success": true,
    "shapeId": "shape_abc123",
    "message": "Rectangle created successfully"
  }
}
```

#### 3. Error Handling

```json
{
  "type": "tool_result",
  "requestId": "req_124",
  "error": {
    "code": "INVALID_PARAMS",
    "message": "Width must be greater than 0",
    "retryable": true
  }
}
```

### Key Design Principles

1. **Domain Agnostic:** Tool abstraction allows any domain
2. **Backward Compatible:** Existing ACP clients can ignore tool capabilities
3. **MCP Compatible:** Tool definitions match MCP format exactly
4. **Secure:** Clients control which tools to expose
5. **Composable:** Multiple MCP servers can be exposed through one ACP client

---

## 🏗️ Architecture

### Full Stack View

```
┌──────────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER (Optional)                                │
│  • A2UI or similar declarative UI protocol                   │
│  • Agent renders visual interfaces                           │
│  • Progress indicators, forms, result display                │
└────────────────────────┬─────────────────────────────────────┘
                         │ (separate concern)
┌────────────────────────▼─────────────────────────────────────┐
│  LOGIC LAYER (ACP + MCP)                                      │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ ACP Protocol Layer                                       │ │
│  │  • Bidirectional communication                          │ │
│  │  • Session management                                   │ │
│  │  • Prompt/response flow                                 │ │
│  │  • Tool discovery & invocation                          │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ MCP Integration Layer                                    │ │
│  │  • Tool definition (MCP format)                         │ │
│  │  • Capability registry                                  │ │
│  │  • Security/approval flows                              │ │
│  │  • Tool execution                                       │ │
│  └─────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────────┐
│  APPLICATION LAYER                                            │
│  • Domain-specific logic                                     │
│  • MCP server implementations                                │
│  • Native capabilities                                       │
└───────────────────────────────────────────────────────────────┘
```

### Component Interaction Flow

```
User Prompt
    │
    ▼
┌─────────────────┐
│  ACP Client     │ ──┐
│  (Application)  │   │
└─────────────────┘   │ 1. Send prompt
    │                 │
    │ Expose tools    │
    ▼                 │
┌─────────────────┐   │
│  MCP Servers    │   │
│  (Domain Tools) │   │
└─────────────────┘   │
                      ▼
              ┌───────────────┐
              │  ACP Server   │
              │  (Agent)      │
              └───────────────┘
                      │
                      │ 2. Discover tools
                      │ 3. Plan actions
                      │
                      ▼
              ┌───────────────┐
              │  LLM/Reasoning│
              └───────────────┘
                      │
                      │ 4. Invoke tools
                      ▼
┌─────────────────────────────────┐
│  invoke_tool("create_shape", …) │
└─────────────────────────────────┘
                      │
                      ▼
              ┌───────────────┐
              │  Tool Result  │
              └───────────────┘
                      │
                      │ 5. Return response
                      ▼
              ┌───────────────┐
              │  User         │
              └───────────────┘
```

### Integration Patterns

#### Pattern 1: Embedded MCP Servers

```typescript
// Application embeds MCP servers directly

import { Server } from "@modelcontextprotocol/sdk/server";
import { AcpClient } from "@agentclientprotocol/sdk";

// Create MCP server with domain tools
const mcpServer = new Server({
  name: "figma-tools",
  version: "1.0.0"
});

mcpServer.tool("create_rectangle", {
  description: "Create a rectangle shape",
  inputSchema: { /* ... */ },
  handler: async (params) => {
    // Figma-specific implementation
    const rect = figma.createRectangle();
    rect.x = params.x;
    rect.y = params.y;
    // ...
    return { id: rect.id };
  }
});

// Integrate with ACP client
const acpClient = new AcpClient();
acpClient.registerMcpServer(mcpServer);
```

#### Pattern 2: External MCP Server Connection

```typescript
// Application connects to external MCP servers

import { McpClient } from "@modelcontextprotocol/sdk/client";
import { AcpClient } from "@agentclientprotocol/sdk";

// Connect to MCP servers
const fsServer = await McpClient.connect("filesystem");
const gitServer = await McpClient.connect("github");
const dbServer = await McpClient.connect("postgres");

// Expose all tools via ACP
const acpClient = new AcpClient();
acpClient.exposeTools([
  ...fsServer.tools,
  ...gitServer.tools,
  ...dbServer.tools
]);
```

#### Pattern 3: Hybrid (Both)

```typescript
// Mix embedded and external servers

const acpClient = new AcpClient();

// Embedded domain-specific tools
acpClient.registerMcpServer(figmaToolsServer);

// External general-purpose tools
acpClient.exposeTools([
  ...filesystemServer.tools,
  ...githubServer.tools
]);
```

---

## 📖 Use Cases

### 1. Figma Design Copilot

**Scenario:** Designer asks agent to create a login form

**Tool Exposure:**
```json
{
  "tools": [
    {"name": "create_frame", ...},
    {"name": "create_text_field", ...},
    {"name": "create_button", ...},
    {"name": "apply_auto_layout", ...},
    {"name": "set_fill_color", ...},
    {"name": "export_png", ...}
  ]
}
```

**Interaction Flow:**
```
User: "Create a modern login form with email, password, and submit button"

Agent thinks:
1. Create frame container
2. Add text fields with labels
3. Add button
4. Apply styling
5. Arrange with auto-layout

Agent executes:
→ invoke_tool("create_frame", {width: 400, height: 500})
→ invoke_tool("create_text_field", {label: "Email", type: "email"})
→ invoke_tool("create_text_field", {label: "Password", type: "password"})
→ invoke_tool("create_button", {label: "Sign In", variant: "primary"})
→ invoke_tool("apply_auto_layout", {direction: "vertical", spacing: 16})

Result: Complete login form rendered in Figma
```

### 2. Excel Data Analysis Copilot

**Scenario:** Analyst asks for insights on sales data

**Tool Exposure:**
```json
{
  "tools": [
    {"name": "read_range", ...},
    {"name": "execute_formula", ...},
    {"name": "create_pivot_table", ...},
    {"name": "create_chart", ...},
    {"name": "apply_conditional_format", ...}
  ]
}
```

**Interaction Flow:**
```
User: "Find anomalies in column C and visualize trends"

Agent executes:
→ invoke_tool("read_range", {range: "C:C"})
→ invoke_tool("execute_formula", {formula: "=STDEV(C:C)"})
→ invoke_tool("create_chart", {type: "line", data: "C:C", ...})
→ invoke_tool("apply_conditional_format", {rule: "above_average", ...})

Result: Anomalies highlighted, trend chart created
```

### 3. VS Code Multi-Tool Workflow

**Scenario:** Developer wants to fix bug and commit

**Tool Exposure:**
```json
{
  "tools": [
    {"name": "read_file", ...},
    {"name": "write_file", ...},
    {"name": "search_codebase", ...},
    {"name": "run_tests", ...},
    {"name": "git_commit", ...},
    {"name": "create_pr", ...}
  ]
}
```

**Interaction Flow:**
```
User: "Fix the null pointer bug in UserService and create a PR"

Agent executes:
→ invoke_tool("search_codebase", {query: "UserService null"})
→ invoke_tool("read_file", {path: "src/UserService.ts"})
→ invoke_tool("write_file", {path: "...", content: "/* fixed */"})
→ invoke_tool("run_tests", {pattern: "UserService.test"})
→ invoke_tool("git_commit", {message: "fix: handle null in UserService"})
→ invoke_tool("create_pr", {title: "Fix null pointer", ...})

Result: Bug fixed, tests pass, PR created
```

### 4. Photoshop Creative Workflow

**Scenario:** Photo editor wants to batch process images

**Tool Exposure:**
```json
{
  "tools": [
    {"name": "open_image", ...},
    {"name": "select_subject", ...},
    {"name": "remove_background", ...},
    {"name": "adjust_levels", ...},
    {"name": "apply_filter", ...},
    {"name": "export_image", ...},
    {"name": "create_action", ...}
  ]
}
```

**Interaction Flow:**
```
User: "Remove backgrounds and enhance colors for all product photos"

Agent executes (for each image):
→ invoke_tool("open_image", {path: "product_001.jpg"})
→ invoke_tool("select_subject", {})
→ invoke_tool("remove_background", {})
→ invoke_tool("adjust_levels", {auto: true})
→ invoke_tool("apply_filter", {name: "vibrance", amount: 20})
→ invoke_tool("export_image", {format: "png", path: "output/..."})

Result: All images processed consistently
```

---

## 🛠️ Implementation Guide

### For Application Developers

#### Step 1: Implement ACP Client

```typescript
import { ClientSideConnection } from "@agentclientprotocol/sdk";

const acpConnection = new ClientSideConnection({
  // Stdio transport (for IDE integration)
  transport: process.stdin,
  output: process.stdout
});

// Handle agent messages
acpConnection.on("prompt", async (prompt) => {
  // Forward to agent, get response
  const response = await processPrompt(prompt);
  return response;
});
```

#### Step 2: Define MCP Tools

```typescript
import { Server } from "@modelcontextprotocol/sdk/server";

const toolServer = new Server({
  name: "my-app-tools",
  version: "1.0.0"
});

// Define each tool your application can perform
toolServer.tool("my_tool_name", {
  description: "What this tool does",
  inputSchema: {
    type: "object",
    properties: {
      param1: { type: "string", description: "..." },
      param2: { type: "number", description: "..." }
    },
    required: ["param1"]
  },
  handler: async (params) => {
    // Your application logic here
    const result = await myAppFunction(params.param1, params.param2);
    return { success: true, result };
  }
});
```

#### Step 3: Bridge ACP and MCP

```typescript
// Register tools with ACP client
acpConnection.advertiseCapabilities({
  tools: toolServer.listTools() // MCP format tools
});

// Handle tool invocation requests from agent
acpConnection.on("invoke_tool", async (request) => {
  const { tool, arguments: args } = request;
  
  // Execute via MCP server
  const result = await toolServer.executeTool(tool, args);
  
  return {
    requestId: request.requestId,
    result
  };
});
```

#### Step 4: Add Security/Approval Layer

```typescript
// Optional: require user approval for sensitive operations
acpConnection.on("invoke_tool", async (request) => {
  const { tool, arguments: args } = request;
  
  // Check if tool requires approval
  if (SENSITIVE_TOOLS.includes(tool)) {
    const approved = await showApprovalDialog({
      tool,
      args,
      description: toolServer.getToolDescription(tool)
    });
    
    if (!approved) {
      return {
        requestId: request.requestId,
        error: {
          code: "USER_DENIED",
          message: "User denied permission"
        }
      };
    }
  }
  
  // Execute tool
  const result = await toolServer.executeTool(tool, args);
  return { requestId: request.requestId, result };
});
```

### For Agent Developers

#### Step 1: Discover Client Tools

```typescript
import { AgentSideConnection } from "@agentclientprotocol/sdk";

const agentConnection = new AgentSideConnection({
  transport: process.stdout,
  input: process.stdin
});

// Listen for client capabilities
let availableTools = [];

agentConnection.on("client_capabilities", (capabilities) => {
  availableTools = capabilities.tools || [];
  console.log(`Client exposed ${availableTools.length} tools`);
});
```

#### Step 2: Invoke Tools as Needed

```typescript
async function processUserPrompt(prompt: string) {
  // Use LLM to decide which tools to call
  const plan = await llm.generatePlan(prompt, availableTools);
  
  for (const step of plan.steps) {
    if (step.type === "tool_call") {
      // Invoke client tool via ACP
      const result = await agentConnection.invokeTool({
        tool: step.toolName,
        arguments: step.arguments
      });
      
      // Use result in next step
      step.result = result;
    }
  }
  
  // Generate final response
  return llm.generateResponse(plan);
}
```

#### Step 3: Handle Tool Results

```typescript
agentConnection.on("tool_result", (result) => {
  if (result.error) {
    // Handle error (retry, fallback, inform user)
    if (result.error.retryable) {
      // Retry with adjusted parameters
    } else {
      // Inform user of failure
    }
  } else {
    // Use result in agent reasoning
    context.addToolResult(result);
  }
});
```

### Integration Examples

#### Figma Plugin Example

```typescript
// figma-plugin.ts
import { ClientSideConnection } from "@agentclientprotocol/sdk";
import { Server } from "@modelcontextprotocol/sdk/server";

// Create MCP server with Figma capabilities
const figmaTools = new Server({name: "figma", version: "1.0.0"});

figmaTools.tool("create_rectangle", {
  description: "Create a rectangle shape",
  inputSchema: {
    type: "object",
    properties: {
      x: {type: "number"},
      y: {type: "number"},
      width: {type: "number"},
      height: {type: "number"},
      fillColor: {type: "string", pattern: "^#[0-9A-Fa-f]{6}$"}
    },
    required: ["x", "y", "width", "height"]
  },
  handler: async (params) => {
    const rect = figma.createRectangle();
    rect.x = params.x;
    rect.y = params.y;
    rect.resize(params.width, params.height);
    
    if (params.fillColor) {
      rect.fills = [{
        type: 'SOLID',
        color: hexToRgb(params.fillColor)
      }];
    }
    
    figma.currentPage.appendChild(rect);
    
    return {
      id: rect.id,
      name: rect.name
    };
  }
});

// Set up ACP connection
const acp = new ClientSideConnection({
  transport: /* stdio or WebSocket */
});

// Advertise Figma tools
acp.advertiseCapabilities({
  tools: figmaTools.listTools()
});

// Handle tool invocations
acp.on("invoke_tool", async (req) => {
  const result = await figmaTools.executeTool(req.tool, req.arguments);
  return { requestId: req.requestId, result };
});
```

#### Excel Add-in Example

```typescript
// excel-addin.ts
import { ClientSideConnection } from "@agentclientprotocol/sdk";

const acpClient = new ClientSideConnection({ /* ... */ });

// Expose Excel capabilities
acpClient.advertiseCapabilities({
  tools: [
    {
      name: "read_range",
      description: "Read values from a range of cells",
      inputSchema: {
        type: "object",
        properties: {
          sheet: {type: "string"},
          range: {type: "string", pattern: "^[A-Z]+[0-9]+:[A-Z]+[0-9]+$"}
        },
        required: ["range"]
      }
    },
    {
      name: "write_range",
      description: "Write values to a range of cells",
      inputSchema: {
        type: "object",
        properties: {
          sheet: {type: "string"},
          range: {type: "string"},
          values: {type: "array"}
        },
        required: ["range", "values"]
      }
    },
    {
      name: "create_chart",
      description: "Create a chart from data range",
      inputSchema: {
        type: "object",
        properties: {
          type: {
            type: "string",
            enum: ["line", "bar", "pie", "scatter"]
          },
          dataRange: {type: "string"},
          title: {type: "string"}
        },
        required: ["type", "dataRange"]
      }
    }
  ]
});

// Handle tool invocations
acpClient.on("invoke_tool", async (request) => {
  switch (request.tool) {
    case "read_range":
      return Excel.run(async (context) => {
        const sheet = context.workbook.worksheets.getActiveWorksheet();
        const range = sheet.getRange(request.arguments.range);
        range.load("values");
        await context.sync();
        return { values: range.values };
      });
      
    case "create_chart":
      return Excel.run(async (context) => {
        const sheet = context.workbook.worksheets.getActiveWorksheet();
        const dataRange = sheet.getRange(request.arguments.dataRange);
        const chart = sheet.charts.add(
          request.arguments.type,
          dataRange,
          Excel.ChartSeriesBy.auto
        );
        chart.title.text = request.arguments.title || "Chart";
        await context.sync();
        return { chartId: chart.id };
      });
      
    // ... more tools
  }
});
```

---

## 🔄 Comparison with A2UI

### Different Layers, Complementary Purposes

| Aspect | **ACP + MCP** | **A2UI** |
|--------|---------------|----------|
| **Purpose** | Logic & capability invocation | Visual presentation |
| **Direction** | Bidirectional | One-way (agent → UI) |
| **Layer** | Application logic | Presentation |
| **Format** | JSON-RPC style | Declarative components |
| **Use Case** | "What can you do?" | "What should I show?" |

### Why Both Are Needed

**ACP + MCP handles:**
- Agent ↔ application communication
- Tool discovery and invocation
- Data operations
- Domain-specific actions

**A2UI handles:**
- Agent → user visual communication
- Progress indicators
- Form collection
- Rich result display
- Interactive UI generation

### Complete Stack

```
┌──────────────────────────────────────────┐
│  A2UI (Presentation Layer)               │
│  • Declarative UI rendering              │
│  • Progress & status                     │
│  • Form collection                       │
│  • Result visualization                  │
└──────────────────────────────────────────┘
              ↕ (separate concern)
┌──────────────────────────────────────────┐
│  ACP + MCP (Logic Layer)                 │
│  • Tool invocation                       │
│  • Capability discovery                  │
│  • Data operations                       │
│  • Application control                   │
└──────────────────────────────────────────┘
```

### Real-World Integration

**Example: Figma Copilot with Both Layers**

```typescript
// User prompt
"Create a dashboard with sales chart and KPIs"

// Logic Layer (ACP + MCP)
Agent → App: invoke_tool("query_sales_data")
App → Agent: {data: [...]}
Agent → App: invoke_tool("create_frame", {width: 1200, height: 800})
Agent → App: invoke_tool("create_chart", {type: "line", data: [...]})

// Presentation Layer (A2UI) - parallel
Agent → UI: {render: ProgressBar("Creating dashboard... 50%")}
Agent → UI: {render: StatusCard("Chart created ✓")}
Agent → UI: {render: CompletionCard("Dashboard ready!")}

// Result: Dashboard created + user sees progress
```

**Key Insight:** 
- Use **ACP+MCP** when agent needs to **DO** something
- Use **A2UI** when agent needs to **SHOW** something
- They work together but solve different problems

---

## 🔮 Future Vision

### Short Term (6-12 months)

**Protocol Extensions:**
- [ ] Add `client_capabilities` message to ACP spec
- [ ] Add `invoke_tool` / `tool_result` messages
- [ ] Publish updated ACP TypeScript SDK
- [ ] Create ACP-MCP bridge library

**Reference Implementations:**
- [ ] VS Code extension with filesystem MCP tools
- [ ] Figma plugin with design MCP tools
- [ ] Excel add-in with data MCP tools
- [ ] Browser extension example

**Ecosystem Growth:**
- [ ] Documentation and best practices
- [ ] Example repositories for common patterns
- [ ] Developer tutorials and guides

### Medium Term (1-2 years)

**Adoption:**
- Multiple IDE integrations (IntelliJ, Vim, Emacs)
- Design tool plugins (Sketch, Canva, Adobe XD)
- Data tool integrations (Tableau, Power BI, Airtable)
- Enterprise app connectors (Salesforce, ServiceNow)

**Standards:**
- ACP extension officially adopted
- Interoperability testing framework
- Certification program for implementations

**Tooling:**
- Visual tool builder for MCP servers
- ACP+MCP debugging tools
- Performance profiling utilities
- Security auditing tools

### Long Term (2-5 years)

**Universal Adoption:**
- ACP+MCP as de facto standard for agent-application integration
- Major platforms supporting natively (Windows, macOS, iOS, Android)
- Cloud services exposing capabilities via ACP+MCP
- Embedded systems with lightweight implementations

**Advanced Features:**
- Multi-agent orchestration patterns
- Federated capability marketplaces
- Cross-application workflows
- Agent-to-agent communication

**Alternative Futures:**

**Scenario A: Convergence**
- ACP+MCP becomes THE universal protocol
- Similar to how LSP unified language tooling
- Single standard across all domains

**Scenario B: Federation**
- Family of domain-specific protocols inspired by ACP+MCP
- DCP (Design Client Protocol) for design tools
- ACP (Analytics Client Protocol) for data tools
- CCP (Creative Client Protocol) for creative tools
- All share common patterns and tooling

**Scenario C: Layered Evolution**
- Lower layer: Universal transport + session management
- Middle layer: Domain-agnostic tool invocation (MCP-style)
- Upper layer: Domain-specific extensions
- Like TCP/IP → HTTP → REST APIs

---

## 📚 References

### Protocols

**Agent Client Protocol (ACP):**
- Website: https://agentclientprotocol.com
- TypeScript SDK: https://github.com/agentclientprotocol/typescript-sdk
- Documentation: https://agentclientprotocol.com/protocol/overview
- API Reference: https://agentclientprotocol.github.io/typescript-sdk

**Model Context Protocol (MCP):**
- Website: https://modelcontextprotocol.io
- Specification: https://spec.modelcontextprotocol.io
- TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk
- Server Repository: https://github.com/modelcontextprotocol/servers

**OpenClaw (A2UI Implementation):**
- Documentation: https://docs.openclaw.ai
- GitHub: https://github.com/openclaw/openclaw
- ACP Integration: https://docs.openclaw.ai/cli/acp

### Related Work

**Language Server Protocol (LSP):**
- Specification: https://microsoft.github.io/language-server-protocol/
- Inspiration for domain-specific standardization

**Anthropic Claude Desktop:**
- MCP integration example
- Desktop app using MCP for tool access

**Zed Editor:**
- ACP implementation reference
- Production IDE with agent integration

### Community

**Discussions:**
- ACP TypeScript SDK Discussions: https://github.com/agentclientprotocol/typescript-sdk/discussions
- MCP Discussions: https://github.com/modelcontextprotocol/specification/discussions

**Contributing:**
- This proposal is open for feedback and contributions
- File issues or PRs for improvements
- Join discussions in respective protocol communities

---

## 🤝 Contributing

This proposal is a living document. Contributions welcome!

**How to contribute:**
1. Fork this repository
2. Create a branch for your changes
3. Submit a pull request with:
   - Clear description of changes
   - Reasoning for proposals
   - Examples when applicable

**Areas needing input:**
- Technical feasibility feedback
- Implementation challenges
- Security considerations
- Use case refinement
- Protocol design details

---

## 📄 License

This proposal document is released under [MIT License](LICENSE).

Protocol implementations should follow their respective licenses:
- ACP: Apache-2.0 (Zed Industries)
- MCP: MIT (Anthropic)

---

**Document Version:** 1.0  
**Last Updated:** 2026-03-07  
**Status:** Proposal / Request for Comments

---

## Appendix: Technical Considerations

### Security

**Trust Boundaries:**
- Client controls which tools to expose
- Agent cannot access unexposed capabilities
- User approval for sensitive operations
- Sandboxed tool execution

**Authentication:**
- Existing ACP auth mechanisms apply
- Additional OAuth for cloud services
- Token-based access for remote tools

**Privacy:**
- No tool data sent to agent unless invoked
- Client-side execution for sensitive operations
- Audit logs for tool invocations

### Performance

**Latency:**
- Tool calls add round-trips
- Mitigation: batch tool calls when possible
- Caching for repeated calls
- Async execution for long operations

**Scalability:**
- Tool discovery O(n) where n = number of tools
- Mitigation: lazy loading, categories
- Streaming for large results

### Compatibility

**Backward Compatibility:**
- Existing ACP clients work without changes
- Tool capabilities are optional extension
- Graceful degradation for old agents

**Forward Compatibility:**
- Versioned tool schemas
- Feature detection
- Protocol version negotiation

---

**Ready to build the future of agent-application integration? Let's make it happen!** 🚀
