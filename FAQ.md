# Frequently Asked Questions

Common questions about the Universal Agent Protocol proposal.

## General Questions

### What is this proposal about?

This proposal extends the Agent Client Protocol (ACP) to support bidirectional tool invocation using the Model Context Protocol (MCP) format. This would make ACP a truly universal protocol for any application to integrate AI agents.

### Why do we need this?

Currently, each application domain needs custom agent integration protocols. By combining ACP (communication) with MCP (tool invocation), we can create one standard that works across all domains—code editors, design tools, data tools, creative apps, and more.

### Is this ready to use?

**No, this is a proposal.** Neither ACP nor MCP currently support this integration pattern. This document proposes extensions to make it possible.

### Who should care about this?

- **Application developers** wanting to add AI copilot features
- **Agent developers** building multi-domain agents
- **Protocol maintainers** (Zed for ACP, Anthropic for MCP)
- **Tool builders** creating MCP servers
- **End users** who want AI assistance across all their tools

---

## Technical Questions

### How does this differ from current ACP?

**Current ACP:**
- Code editor specific
- Fixed command set (read_file, write_file, etc.)
- Agent provides capabilities

**Proposed ACP+MCP:**
- Domain agnostic
- Dynamic tool discovery
- **Client exposes capabilities** via MCP format
- Agent invokes client tools as needed

### Why MCP specifically?

1. **Existing standard**: MCP already defines tool schemas
2. **Growing ecosystem**: Many MCP servers already exist
3. **Well-designed**: Good tool definition format
4. **Complementary**: MCP and ACP solve different problems

### Can't we just use MCP alone?

MCP defines tool interfaces but not client-agent communication. ACP provides:
- Session management
- Prompt/response flow
- Connection lifecycle
- Multiple transport options

You need both: ACP for communication layer, MCP for tool format.

### How is this different from function calling?

Function calling (OpenAI, Claude) is **agent→tool** (one direction).

This proposal adds **client→agent** tool exposure:
- Application tells agent what it can do
- Agent discovers capabilities dynamically
- Truly bidirectional

### What about security?

The proposal includes:
- Client controls which tools to expose
- User approval for sensitive operations
- Sandboxed tool execution
- Audit logging
- Token-based auth for remote tools

See the Security section in README.md for details.

---

## Implementation Questions

### Do I need to rewrite my application?

No! Integration steps:
1. Implement ACP client (stdio or WebSocket)
2. Define your tools in MCP format
3. Bridge them together

Existing MCP servers can be reused.

### Can I use existing MCP servers?

**Yes!** That's a key benefit. You can:
- Connect to external MCP servers
- Expose their tools via your ACP client
- Mix with embedded domain-specific tools

### What languages are supported?

ACP and MCP have TypeScript SDKs. Community implementations exist for:
- Python
- Go
- Rust
- Java

The protocols are language-agnostic (JSON over stdio/WebSocket).

### Do I need to run a separate process?

**No.** You can:
- Embed MCP server in your application process
- Connect to external MCP servers
- Or both (hybrid approach)

### How do I handle errors?

Tool invocations return standardized errors:
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "retryable": true/false
  }
}
```

Agents can retry with adjusted parameters if `retryable: true`.

---

## Comparison Questions

### ACP vs MCP vs AG-UI?

**Different protocols for different purposes:**

| Protocol | Layer | Purpose |
|----------|-------|---------|
| **ACP** | Communication | IDE ↔ Agent (code editing) |
| **MCP** | Tool/Data | Agent ↔ Tools & Data |
| **AG-UI** | UI/UX | Agent ↔ User-facing Apps |

**This proposal:** Extend ACP with MCP-style tool invocation to make it universal (any app type).

**AG-UI** focuses on user-facing copilot experiences with rich UI.
**ACP+MCP** focuses on capability exposure for any application (including CLIs, servers, non-UI tools).

### Why not just extend MCP?

MCP is tool-focused, not communication-focused. It doesn't handle:
- Session lifecycle
- Multiple transports
- Prompts and responses
- Connection management

ACP already solves these. Better to combine strengths.

### Why not create a new protocol?

**Leverage existing work:**
- ACP: 2+ years of production use (Zed)
- MCP: Active development by Anthropic
- Both have growing ecosystems

Building on standards is better than starting from scratch.

### How does this compare to LSP?

**Similarities:**
- LSP unified language servers
- This could unify agent-application integration
- Both use JSON-RPC-style communication

**Differences:**
- LSP is synchronous, domain-specific (code)
- This is asynchronous, domain-agnostic
- LSP is editor→server; this is bidirectional

---

## Adoption Questions

### Who needs to adopt this?

**For success:**
1. ACP maintainers add tool invocation support
2. Application developers integrate ACP+MCP
3. Agent developers support client tool invocation

**Network effects:** More apps = more useful agents = more apps

### What's the adoption timeline?

**Speculative:**
- Short term (6-12 months): Protocol extensions, reference implementations
- Medium term (1-2 years): IDE, design tool, data tool integrations
- Long term (2-5 years): Standard for agent-application integration

### Is this backwards compatible?

**Yes.** Proposed extensions are opt-in:
- Existing ACP clients work without changes
- Tool capabilities are optional
- Graceful degradation for old agents

### What if ACP/MCP maintainers don't adopt this?

**Alternative paths:**
1. Community fork with extensions
2. Inspired protocols (DCP, ACP variants)
3. Demonstrate value with proof-of-concept

Compelling use cases drive adoption.

---

## Use Case Questions

### What applications would benefit?

**Any application with programmable capabilities:**
- Design tools (Figma, Sketch, Canva)
- Data tools (Excel, Tableau, Airtable)
- Creative tools (Photoshop, Premiere, Logic)
- IDEs (VS Code, IntelliJ, Vim)
- Productivity (Notion, Obsidian, Roam)
- Enterprise (Salesforce, ServiceNow)

Essentially: **everything**.

### Can this work on mobile?

**Yes!** Transport options include:
- WebSocket (works on mobile)
- HTTP/REST (works on mobile)
- IPC (native mobile apps)

AG-UI and similar protocols work on mobile platforms.

### Can this work in the browser?

**Yes!** Browser extensions can:
- Implement ACP client
- Expose webpage capabilities as tools
- Use WebSocket transport

Example: Browser automation tools.

### What about offline applications?

**Yes!** Local agent + local application:
- Stdio transport (local process)
- IPC (inter-process communication)
- No internet required

### Can multiple agents share one application?

**Yes!** Each agent gets its own ACP session:
- Separate tool discovery
- Isolated contexts
- Session management in ACP

---

## Development Questions

### Where do I start?

**For application developers:**
1. Read the [Implementation Guide](README.md#implementation-guide)
2. Check [Examples](EXAMPLES.md)
3. Start with a simple tool exposure

**For agent developers:**
1. Understand tool discovery flow
2. Implement tool invocation
3. Handle tool results gracefully

### Are there example implementations?

**Currently:** This is a proposal, so no production implementations yet.

**When available:**
- Reference implementations will be provided
- Example integrations (Figma, Excel, VS Code)
- SDK extensions for ACP+MCP bridge

### How do I test this?

**Development approach:**
1. Mock ACP agent for testing
2. Test tool discovery
3. Test tool invocation
4. Test error handling
5. Test security/approval flows

**Testing tools** (to be created):
- ACP+MCP test harness
- Mock agent for development
- Tool validation utilities

### Where can I contribute?

**This repository:**
- Open issues for questions
- Submit PRs for improvements
- Share implementation feedback

**Protocol repositories:**
- ACP: https://github.com/agentclientprotocol/typescript-sdk
- MCP: https://github.com/modelcontextprotocol/specification

### How do I give feedback?

1. **GitHub Issues** on this repository
2. **Discussions** in ACP/MCP communities
3. **Pull Requests** with improvements
4. **Real-world prototypes** showing feasibility

---

## Future Questions

### What happens after this proposal?

**Ideal path:**
1. Community feedback and refinement
2. Proof-of-concept implementations
3. Protocol maintainers review and adopt
4. SDK updates with ACP+MCP bridge
5. Application integrations begin
6. Ecosystem growth

### Could this become a standard?

**Possibly.** Success requires:
- Technical feasibility (proven)
- Developer adoption (needs work)
- Protocol maintainer buy-in (TBD)
- Compelling use cases (many exist)

Standards emerge from proven value.

### What if something better comes along?

**Great!** This proposal is about:
- Solving the universality problem
- Demonstrating what's possible
- Starting the conversation

Better solutions are welcome. Progress is the goal.

### Will this replace proprietary solutions?

**Eventually, maybe.** Open standards tend to win long-term when they:
- Provide real value
- Have ecosystem support
- Reduce friction

GitHub Copilot, Cursor, etc. might adopt or compete.

### How does this fit with AI progress?

**Future-proof principles:**
- Tool-based architectures (agents use tools)
- Capability discovery (dynamic, not hardcoded)
- Modular design (swap agents, keep apps)

As AI improves, capable agents need capable tools.

---

## Philosophical Questions

### Why does this matter?

**AI-native applications** are the future. But today:
- Each app needs custom integration
- Agents are siloed to specific domains
- Users can't bring their agent everywhere

**This proposal enables:**
- Any app can integrate any agent
- Users control their AI experience
- Innovation at edges (apps and agents independently)

### Isn't this just making everything agentic?

**No.** It's giving applications an **AI API**—a standard way to expose capabilities. Applications remain in control:
- They decide what to expose
- They handle security
- They own the UX

Agents are powerful assistants, not replacements.

### What about AI safety?

**Good question.** This proposal includes:
- Human-in-the-loop for sensitive operations
- Audit trails for all tool invocations
- Sandboxed execution
- Rate limiting and quotas

**But:** As capabilities grow, safety considerations must too. This is an ongoing conversation.

### Could this be abused?

**Any powerful tool can be.** Mitigations:
- User approval for sensitive tools
- Scoped permissions (principle of least privilege)
- Audit logging
- Revocable access

Security is a priority, not an afterthought.

---

## Meta Questions

### Who wrote this proposal?

This proposal emerged from a conversation about making ACP universal by integrating MCP for tool invocation. It synthesizes ideas from:
- ACP (Zed Industries)
- MCP (Anthropic)
- AG-UI (CopilotKit)
- Language Server Protocol pattern

### How accurate is this?

**Protocol details:** Accurate based on current ACP and MCP specs.
**Proposal specifics:** Speculative—this is a design proposal, not implemented yet.
**Timeline/adoption:** Educated guesses, not commitments.

### Can I share this?

**Yes!** This proposal is open for discussion. Share widely:
- Link to this repository
- Discuss in communities
- Build prototypes
- Provide feedback

### How do I cite this?

```
Universal Agent Protocol: Making ACP Universal Through MCP Integration
https://github.com/[your-org]/universal-agent-protocol
Version 1.0 (2026-03-07)
```

### Will this be maintained?

**For now:** Yes, as a proposal and reference.

**Long term:** Depends on:
- Community interest
- Protocol maintainer feedback
- Implementation progress

Contributions welcome!

---

## Still Have Questions?

**Open an issue on this repository** or **start a discussion**:
- GitHub Issues: For specific technical questions
- Discussions: For broader conversation
- ACP Community: https://github.com/agentclientprotocol/typescript-sdk/discussions
- MCP Community: https://github.com/modelcontextprotocol/specification/discussions

We're building this together! 🚀
