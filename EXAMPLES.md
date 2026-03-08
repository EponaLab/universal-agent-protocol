# Examples & Code Samples

Real-world implementation examples for the Universal Agent Protocol.

## Table of Contents

1. [Basic Integration](#basic-integration)
2. [Figma Plugin Example](#figma-plugin-example)
3. [Excel Add-in Example](#excel-add-in-example)
4. [VS Code Extension Example](#vs-code-extension-example)
5. [Multi-Domain Integration](#multi-domain-integration)
6. [Security & Approval Patterns](#security--approval-patterns)

---

## Basic Integration

### Minimal ACP Client with MCP Tools

```typescript
import { ClientSideConnection } from "@agentclientprotocol/sdk";
import { Server } from "@modelcontextprotocol/sdk/server";

// 1. Create MCP server with your tools
const toolServer = new Server({
  name: "my-app-tools",
  version: "1.0.0"
});

// 2. Define tools
toolServer.tool("hello_world", {
  description: "Returns a greeting",
  inputSchema: {
    type: "object",
    properties: {
      name: { type: "string", description: "Name to greet" }
    },
    required: ["name"]
  },
  handler: async (params) => {
    return { message: `Hello, ${params.name}!` };
  }
});

// 3. Create ACP connection
const acpClient = new ClientSideConnection({
  transport: process.stdin,
  output: process.stdout
});

// 4. Advertise tools when connected
acpClient.on("connected", () => {
  acpClient.advertiseCapabilities({
    tools: toolServer.listTools()
  });
});

// 5. Handle tool invocations
acpClient.on("invoke_tool", async (request) => {
  try {
    const result = await toolServer.executeTool(
      request.tool,
      request.arguments
    );
    
    return {
      requestId: request.requestId,
      result
    };
  } catch (error) {
    return {
      requestId: request.requestId,
      error: {
        code: "EXECUTION_ERROR",
        message: error.message,
        retryable: false
      }
    };
  }
});

// 6. Start
acpClient.start();
```

---

## Figma Plugin Example

### Complete Figma Copilot Integration

```typescript
// figma-plugin/main.ts

import { ClientSideConnection } from "@agentclientprotocol/sdk";
import { Server } from "@modelcontextprotocol/sdk/server";

// ============================================================================
// MCP Server Setup
// ============================================================================

const figmaTools = new Server({
  name: "figma-design-tools",
  version: "1.0.0",
  description: "Figma design manipulation tools"
});

// Tool: Create Rectangle
figmaTools.tool("create_rectangle", {
  description: "Create a rectangle shape on the canvas",
  inputSchema: {
    type: "object",
    properties: {
      x: { type: "number", description: "X position" },
      y: { type: "number", description: "Y position" },
      width: { type: "number", description: "Width" },
      height: { type: "number", description: "Height" },
      name: { type: "string", description: "Shape name" },
      fillColor: {
        type: "string",
        description: "Fill color in hex format",
        pattern: "^#[0-9A-Fa-f]{6}$"
      },
      cornerRadius: {
        type: "number",
        description: "Corner radius (0 for sharp corners)"
      }
    },
    required: ["x", "y", "width", "height"]
  },
  handler: async (params) => {
    const rect = figma.createRectangle();
    
    rect.x = params.x;
    rect.y = params.y;
    rect.resize(params.width, params.height);
    
    if (params.name) {
      rect.name = params.name;
    }
    
    if (params.fillColor) {
      const color = hexToRgb(params.fillColor);
      rect.fills = [{
        type: 'SOLID',
        color: color
      }];
    }
    
    if (params.cornerRadius !== undefined) {
      rect.cornerRadius = params.cornerRadius;
    }
    
    figma.currentPage.appendChild(rect);
    figma.viewport.scrollAndZoomIntoView([rect]);
    
    return {
      id: rect.id,
      name: rect.name,
      type: rect.type
    };
  }
});

// Tool: Create Text
figmaTools.tool("create_text", {
  description: "Create a text layer",
  inputSchema: {
    type: "object",
    properties: {
      x: { type: "number" },
      y: { type: "number" },
      text: { type: "string", description: "Text content" },
      fontSize: { type: "number", description: "Font size in pixels" },
      fontWeight: {
        type: "string",
        enum: ["normal", "bold", "light"],
        description: "Font weight"
      },
      color: { type: "string", pattern: "^#[0-9A-Fa-f]{6}$" }
    },
    required: ["x", "y", "text"]
  },
  handler: async (params) => {
    // Load font (async)
    await figma.loadFontAsync({ family: "Inter", style: "Regular" });
    
    const text = figma.createText();
    text.x = params.x;
    text.y = params.y;
    text.characters = params.text;
    
    if (params.fontSize) {
      text.fontSize = params.fontSize;
    }
    
    if (params.color) {
      const color = hexToRgb(params.color);
      text.fills = [{
        type: 'SOLID',
        color: color
      }];
    }
    
    figma.currentPage.appendChild(text);
    
    return {
      id: text.id,
      name: text.name,
      characters: text.characters
    };
  }
});

// Tool: Apply Auto Layout
figmaTools.tool("apply_auto_layout", {
  description: "Apply auto layout to a frame",
  inputSchema: {
    type: "object",
    properties: {
      nodeId: { type: "string", description: "Node ID to apply layout to" },
      direction: {
        type: "string",
        enum: ["horizontal", "vertical"],
        description: "Layout direction"
      },
      spacing: { type: "number", description: "Spacing between items" },
      padding: { type: "number", description: "Padding inside frame" }
    },
    required: ["nodeId", "direction"]
  },
  handler: async (params) => {
    const node = figma.getNodeById(params.nodeId);
    
    if (!node || node.type !== "FRAME") {
      throw new Error("Node not found or not a frame");
    }
    
    const frame = node as FrameNode;
    frame.layoutMode = params.direction === "horizontal" ? "HORIZONTAL" : "VERTICAL";
    
    if (params.spacing !== undefined) {
      frame.itemSpacing = params.spacing;
    }
    
    if (params.padding !== undefined) {
      frame.paddingLeft = params.padding;
      frame.paddingRight = params.padding;
      frame.paddingTop = params.padding;
      frame.paddingBottom = params.padding;
    }
    
    return {
      id: frame.id,
      layoutMode: frame.layoutMode,
      itemSpacing: frame.itemSpacing
    };
  }
});

// Tool: Export Selection
figmaTools.tool("export_selection", {
  description: "Export selected nodes as PNG",
  inputSchema: {
    type: "object",
    properties: {
      nodeIds: {
        type: "array",
        items: { type: "string" },
        description: "Node IDs to export"
      },
      scale: {
        type: "number",
        description: "Export scale (1 = 1x, 2 = 2x, etc.)",
        default: 1
      }
    },
    required: ["nodeIds"]
  },
  handler: async (params) => {
    const nodes = params.nodeIds
      .map(id => figma.getNodeById(id))
      .filter(node => node !== null);
    
    if (nodes.length === 0) {
      throw new Error("No valid nodes found");
    }
    
    const exports = await Promise.all(
      nodes.map(async (node) => {
        const bytes = await node.exportAsync({
          format: "PNG",
          constraint: { type: "SCALE", value: params.scale || 1 }
        });
        
        return {
          nodeId: node.id,
          name: node.name,
          bytes: Array.from(bytes),
          size: bytes.length
        };
      })
    );
    
    return { exports };
  }
});

// Helper: Convert hex to RGB
function hexToRgb(hex: string): RGB {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result ? {
    r: parseInt(result[1], 16) / 255,
    g: parseInt(result[2], 16) / 255,
    b: parseInt(result[3], 16) / 255
  } : { r: 0, g: 0, b: 0 };
}

// ============================================================================
// ACP Integration
// ============================================================================

const acpConnection = new ClientSideConnection({
  transport: process.stdin,
  output: process.stdout
});

// Advertise capabilities
acpConnection.on("connected", () => {
  console.log("[Figma Plugin] Connected to agent");
  
  acpConnection.advertiseCapabilities({
    name: "Figma Design Copilot",
    version: "1.0.0",
    tools: figmaTools.listTools()
  });
});

// Handle tool invocations
acpConnection.on("invoke_tool", async (request) => {
  console.log(`[Figma Plugin] Tool invoked: ${request.tool}`);
  
  try {
    const result = await figmaTools.executeTool(
      request.tool,
      request.arguments
    );
    
    console.log(`[Figma Plugin] Tool succeeded: ${request.tool}`);
    
    return {
      requestId: request.requestId,
      result
    };
    
  } catch (error) {
    console.error(`[Figma Plugin] Tool failed: ${request.tool}`, error);
    
    return {
      requestId: request.requestId,
      error: {
        code: "TOOL_EXECUTION_ERROR",
        message: error.message,
        retryable: false
      }
    };
  }
});

// Start connection
acpConnection.start();

console.log("[Figma Plugin] ACP client started");
```

---

## Excel Add-in Example

### Excel Data Analysis Copilot

```typescript
// excel-addin/taskpane.ts

import { ClientSideConnection } from "@agentclientprotocol/sdk";
import { Server } from "@modelcontextprotocol/sdk/server";

// ============================================================================
// MCP Server for Excel Operations
// ============================================================================

const excelTools = new Server({
  name: "excel-data-tools",
  version: "1.0.0"
});

// Tool: Read Range
excelTools.tool("read_range", {
  description: "Read values from a cell range",
  inputSchema: {
    type: "object",
    properties: {
      sheet: { type: "string", description: "Sheet name (optional)" },
      range: {
        type: "string",
        description: "Range address (e.g., 'A1:B10')",
        pattern: "^[A-Z]+[0-9]+:[A-Z]+[0-9]+$"
      }
    },
    required: ["range"]
  },
  handler: async (params) => {
    return Excel.run(async (context) => {
      const sheet = params.sheet
        ? context.workbook.worksheets.getItem(params.sheet)
        : context.workbook.worksheets.getActiveWorksheet();
      
      const range = sheet.getRange(params.range);
      range.load(["values", "formulas", "numberFormat"]);
      
      await context.sync();
      
      return {
        values: range.values,
        formulas: range.formulas,
        formats: range.numberFormat
      };
    });
  }
});

// Tool: Write Range
excelTools.tool("write_range", {
  description: "Write values to a cell range",
  inputSchema: {
    type: "object",
    properties: {
      sheet: { type: "string" },
      range: { type: "string", pattern: "^[A-Z]+[0-9]+(:[A-Z]+[0-9]+)?$" },
      values: {
        type: "array",
        description: "2D array of values",
        items: { type: "array" }
      }
    },
    required: ["range", "values"]
  },
  handler: async (params) => {
    return Excel.run(async (context) => {
      const sheet = params.sheet
        ? context.workbook.worksheets.getItem(params.sheet)
        : context.workbook.worksheets.getActiveWorksheet();
      
      const range = sheet.getRange(params.range);
      range.values = params.values;
      
      await context.sync();
      
      return { success: true, range: params.range };
    });
  }
});

// Tool: Create Chart
excelTools.tool("create_chart", {
  description: "Create a chart from data range",
  inputSchema: {
    type: "object",
    properties: {
      type: {
        type: "string",
        enum: ["line", "bar", "column", "pie", "scatter", "area"],
        description: "Chart type"
      },
      dataRange: { type: "string", description: "Data range for chart" },
      title: { type: "string", description: "Chart title" },
      xAxisTitle: { type: "string" },
      yAxisTitle: { type: "string" }
    },
    required: ["type", "dataRange"]
  },
  handler: async (params) => {
    return Excel.run(async (context) => {
      const sheet = context.workbook.worksheets.getActiveWorksheet();
      const dataRange = sheet.getRange(params.dataRange);
      
      const chartType = mapChartType(params.type);
      
      const chart = sheet.charts.add(
        chartType,
        dataRange,
        Excel.ChartSeriesBy.auto
      );
      
      if (params.title) {
        chart.title.text = params.title;
      }
      
      if (params.xAxisTitle) {
        chart.axes.categoryAxis.title.text = params.xAxisTitle;
      }
      
      if (params.yAxisTitle) {
        chart.axes.valueAxis.title.text = params.yAxisTitle;
      }
      
      await context.sync();
      
      return {
        chartId: chart.id,
        type: params.type,
        title: chart.title.text
      };
    });
  }
});

// Tool: Execute Formula
excelTools.tool("execute_formula", {
  description: "Execute an Excel formula and return result",
  inputSchema: {
    type: "object",
    properties: {
      formula: {
        type: "string",
        description: "Excel formula (e.g., '=SUM(A1:A10)')"
      }
    },
    required: ["formula"]
  },
  handler: async (params) => {
    return Excel.run(async (context) => {
      const sheet = context.workbook.worksheets.getActiveWorksheet();
      
      // Use a temporary cell to evaluate formula
      const tempCell = sheet.getRange("ZZ1");
      tempCell.formulas = [[params.formula]];
      tempCell.load("values");
      
      await context.sync();
      
      const result = tempCell.values[0][0];
      
      // Clear temp cell
      tempCell.clear();
      await context.sync();
      
      return { result };
    });
  }
});

function mapChartType(type: string): Excel.ChartType {
  const mapping = {
    line: Excel.ChartType.line,
    bar: Excel.ChartType.barClustered,
    column: Excel.ChartType.columnClustered,
    pie: Excel.ChartType.pie,
    scatter: Excel.ChartType.xyscatter,
    area: Excel.ChartType.area
  };
  return mapping[type] || Excel.ChartType.columnClustered;
}

// ============================================================================
// ACP Integration
// ============================================================================

let acpClient: ClientSideConnection | null = null;

export function initializeACP() {
  acpClient = new ClientSideConnection({
    transport: process.stdin,
    output: process.stdout
  });
  
  acpClient.on("connected", () => {
    console.log("[Excel Add-in] Connected to agent");
    
    acpClient!.advertiseCapabilities({
      name: "Excel Data Copilot",
      version: "1.0.0",
      tools: excelTools.listTools()
    });
  });
  
  acpClient.on("invoke_tool", async (request) => {
    try {
      const result = await excelTools.executeTool(
        request.tool,
        request.arguments
      );
      
      return {
        requestId: request.requestId,
        result
      };
    } catch (error) {
      return {
        requestId: request.requestId,
        error: {
          code: "EXCEL_ERROR",
          message: error.message,
          retryable: true
        }
      };
    }
  });
  
  acpClient.start();
}

// Initialize when add-in loads
Office.onReady(() => {
  initializeACP();
});
```

---

## VS Code Extension Example

### Multi-Tool Development Copilot

```typescript
// vscode-extension/extension.ts

import * as vscode from 'vscode';
import { ClientSideConnection } from "@agentclientprotocol/sdk";
import { Client as McpClient } from "@modelcontextprotocol/sdk/client";

let acpConnection: ClientSideConnection | null = null;

export async function activate(context: vscode.ExtensionContext) {
  
  // ========================================================================
  // Connect to External MCP Servers
  // ========================================================================
  
  const filesystemServer = await McpClient.connect({
    command: "npx",
    args: ["-y", "@modelcontextprotocol/server-filesystem", process.cwd()]
  });
  
  const githubServer = await McpClient.connect({
    command: "npx",
    args: ["-y", "@modelcontextprotocol/server-github"]
  });
  
  // ========================================================================
  // VS Code Specific Tools
  // ========================================================================
  
  const vscodeTools = [
    {
      name: "open_file",
      description: "Open a file in the editor",
      inputSchema: {
        type: "object",
        properties: {
          path: { type: "string" }
        },
        required: ["path"]
      }
    },
    {
      name: "run_task",
      description: "Run a VS Code task",
      inputSchema: {
        type: "object",
        properties: {
          taskName: { type: "string" }
        },
        required: ["taskName"]
      }
    },
    {
      name: "show_diagnostics",
      description: "Get diagnostics (errors/warnings) for a file",
      inputSchema: {
        type: "object",
        properties: {
          path: { type: "string" }
        },
        required: ["path"]
      }
    }
  ];
  
  // ========================================================================
  // ACP Setup
  // ========================================================================
  
  acpConnection = new ClientSideConnection({
    transport: process.stdin,
    output: process.stdout
  });
  
  // Combine all tools
  const allTools = [
    ...filesystemServer.listTools(),
    ...githubServer.listTools(),
    ...vscodeTools
  ];
  
  acpConnection.on("connected", () => {
    acpConnection!.advertiseCapabilities({
      name: "VS Code Development Copilot",
      version: "1.0.0",
      tools: allTools
    });
  });
  
  // Tool invocation handler
  acpConnection.on("invoke_tool", async (request) => {
    try {
      let result;
      
      // Route to appropriate handler
      if (request.tool.startsWith("fs_")) {
        result = await filesystemServer.executeTool(
          request.tool,
          request.arguments
        );
      } else if (request.tool.startsWith("github_")) {
        result = await githubServer.executeTool(
          request.tool,
          request.arguments
        );
      } else {
        // VS Code specific tools
        result = await handleVSCodeTool(request.tool, request.arguments);
      }
      
      return {
        requestId: request.requestId,
        result
      };
      
    } catch (error) {
      return {
        requestId: request.requestId,
        error: {
          code: "TOOL_ERROR",
          message: error.message,
          retryable: false
        }
      };
    }
  });
  
  acpConnection.start();
}

async function handleVSCodeTool(tool: string, args: any) {
  switch (tool) {
    case "open_file":
      const doc = await vscode.workspace.openTextDocument(args.path);
      await vscode.window.showTextDocument(doc);
      return { opened: true, path: args.path };
      
    case "run_task":
      const tasks = await vscode.tasks.fetchTasks();
      const task = tasks.find(t => t.name === args.taskName);
      if (!task) {
        throw new Error(`Task not found: ${args.taskName}`);
      }
      await vscode.tasks.executeTask(task);
      return { executed: true, task: args.taskName };
      
    case "show_diagnostics":
      const uri = vscode.Uri.file(args.path);
      const diagnostics = vscode.languages.getDiagnostics(uri);
      return {
        path: args.path,
        diagnostics: diagnostics.map(d => ({
          severity: d.severity,
          message: d.message,
          range: {
            start: { line: d.range.start.line, character: d.range.start.character },
            end: { line: d.range.end.line, character: d.range.end.character }
          }
        }))
      };
      
    default:
      throw new Error(`Unknown tool: ${tool}`);
  }
}

export function deactivate() {
  if (acpConnection) {
    acpConnection.stop();
  }
}
```

---

## Multi-Domain Integration

### Application with Multiple Capability Sources

```typescript
import { ClientSideConnection } from "@agentclientprotocol/sdk";
import { Server } from "@modelcontextprotocol/sdk/server";
import { Client as McpClient } from "@modelcontextprotocol/sdk/client";

class UniversalCopilotApp {
  private acpClient: ClientSideConnection;
  private embeddedTools: Server;
  private externalServers: Map<string, any> = new Map();
  
  async initialize() {
    // 1. Create embedded MCP server for app-specific tools
    this.embeddedTools = new Server({
      name: "app-specific-tools",
      version: "1.0.0"
    });
    
    this.defineEmbeddedTools();
    
    // 2. Connect to external MCP servers
    await this.connectExternalServers();
    
    // 3. Setup ACP client
    this.setupACP();
  }
  
  private defineEmbeddedTools() {
    // App-specific domain tools
    this.embeddedTools.tool("domain_action_1", {
      description: "Perform domain-specific action",
      inputSchema: { /* ... */ },
      handler: async (params) => {
        // Your app logic
        return { success: true };
      }
    });
    
    // Add more embedded tools...
  }
  
  private async connectExternalServers() {
    // Filesystem
    const fsServer = await McpClient.connect({
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-filesystem", "./workspace"]
    });
    this.externalServers.set("filesystem", fsServer);
    
    // GitHub
    const ghServer = await McpClient.connect({
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-github"]
    });
    this.externalServers.set("github", ghServer);
    
    // Database
    const dbServer = await McpClient.connect({
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-postgres"]
    });
    this.externalServers.set("database", dbServer);
  }
  
  private setupACP() {
    this.acpClient = new ClientSideConnection({
      transport: process.stdin,
      output: process.stdout
    });
    
    // Aggregate all tools
    this.acpClient.on("connected", () => {
      const allTools = [
        ...this.embeddedTools.listTools(),
        ...this.getExternalTools()
      ];
      
      this.acpClient.advertiseCapabilities({
        name: "Universal Copilot",
        version: "1.0.0",
        tools: allTools,
        metadata: {
          embeddedToolCount: this.embeddedTools.listTools().length,
          externalServerCount: this.externalServers.size
        }
      });
    });
    
    // Route tool invocations
    this.acpClient.on("invoke_tool", async (request) => {
      return this.routeToolInvocation(request);
    });
    
    this.acpClient.start();
  }
  
  private getExternalTools() {
    const tools = [];
    for (const [name, server] of this.externalServers.entries()) {
      tools.push(...server.listTools());
    }
    return tools;
  }
  
  private async routeToolInvocation(request: any) {
    try {
      let result;
      
      // Check embedded tools first
      const embeddedToolNames = this.embeddedTools
        .listTools()
        .map(t => t.name);
      
      if (embeddedToolNames.includes(request.tool)) {
        result = await this.embeddedTools.executeTool(
          request.tool,
          request.arguments
        );
      } else {
        // Route to external server
        result = await this.executeExternalTool(
          request.tool,
          request.arguments
        );
      }
      
      return {
        requestId: request.requestId,
        result
      };
      
    } catch (error) {
      return {
        requestId: request.requestId,
        error: {
          code: "ROUTING_ERROR",
          message: error.message,
          retryable: false
        }
      };
    }
  }
  
  private async executeExternalTool(toolName: string, args: any) {
    for (const [serverName, server] of this.externalServers.entries()) {
      const tools = server.listTools();
      if (tools.some(t => t.name === toolName)) {
        return await server.executeTool(toolName, args);
      }
    }
    
    throw new Error(`Tool not found: ${toolName}`);
  }
}

// Usage
const app = new UniversalCopilotApp();
app.initialize();
```

---

## Security & Approval Patterns

### User Approval for Sensitive Operations

```typescript
import { ClientSideConnection } from "@agentclientprotocol/sdk";

const SENSITIVE_OPERATIONS = [
  "delete_file",
  "execute_command",
  "make_api_call",
  "send_email"
];

class SecureAcpClient {
  private acpClient: ClientSideConnection;
  private approvalCache: Map<string, boolean> = new Map();
  
  constructor() {
    this.acpClient = new ClientSideConnection({
      transport: process.stdin,
      output: process.stdout
    });
    
    this.setupToolInvocationHandler();
  }
  
  private setupToolInvocationHandler() {
    this.acpClient.on("invoke_tool", async (request) => {
      // Check if operation is sensitive
      if (SENSITIVE_OPERATIONS.includes(request.tool)) {
        const approved = await this.requestUserApproval(request);
        
        if (!approved) {
          return {
            requestId: request.requestId,
            error: {
              code: "USER_DENIED",
              message: "User denied permission for this operation",
              retryable: false
            }
          };
        }
      }
      
      // Execute tool
      try {
        const result = await this.executeTool(request);
        return {
          requestId: request.requestId,
          result
        };
      } catch (error) {
        return {
          requestId: request.requestId,
          error: {
            code: "EXECUTION_ERROR",
            message: error.message,
            retryable: false
          }
        };
      }
    });
  }
  
  private async requestUserApproval(request: any): Promise<boolean> {
    // Check cache first
    const cacheKey = `${request.tool}:${JSON.stringify(request.arguments)}`;
    if (this.approvalCache.has(cacheKey)) {
      return this.approvalCache.get(cacheKey)!;
    }
    
    // Show approval dialog
    const approved = await this.showApprovalDialog({
      tool: request.tool,
      arguments: request.arguments,
      description: this.getToolDescription(request.tool)
    });
    
    // Optionally cache approval
    if (approved && await this.askToRemember()) {
      this.approvalCache.set(cacheKey, true);
    }
    
    return approved;
  }
  
  private async showApprovalDialog(options: any): Promise<boolean> {
    // Platform-specific dialog implementation
    // This is a placeholder - implement based on your platform
    
    console.log("\n" + "=".repeat(60));
    console.log("APPROVAL REQUIRED");
    console.log("=".repeat(60));
    console.log(`Tool: ${options.tool}`);
    console.log(`Description: ${options.description}`);
    console.log(`Arguments: ${JSON.stringify(options.arguments, null, 2)}`);
    console.log("=".repeat(60));
    
    // In a real implementation, show a UI dialog
    // For CLI: use readline
    const readline = require('readline');
    const rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });
    
    return new Promise((resolve) => {
      rl.question('\nApprove this operation? (y/n): ', (answer: string) => {
        rl.close();
        resolve(answer.toLowerCase() === 'y');
      });
    });
  }
  
  private async askToRemember(): Promise<boolean> {
    // Ask if user wants to remember this approval
    // Implementation similar to showApprovalDialog
    return false; // Placeholder
  }
  
  private getToolDescription(toolName: string): string {
    // Get tool description from MCP server
    // Placeholder implementation
    return `Execute ${toolName}`;
  }
  
  private async executeTool(request: any): Promise<any> {
    // Execute the tool via MCP server
    // Placeholder implementation
    return { success: true };
  }
}

// Usage
const secureClient = new SecureAcpClient();
```

---

**These examples demonstrate real-world integration patterns. Adapt them to your specific application needs!**
