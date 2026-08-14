# Serverless Architect: Universal Agent Skill (Enterprise Advisory Edition)

A vendor-agnostic, enterprise-grade agent skill for diagnosing, selecting, and architecting serverless hosting platforms across **Amazon Web Services (AWS)** and **Google Cloud Platform (GCP)**.

Designed to guide product teams, platform engineers, and innovation leads through turning "vibe-coded" prototypes, full-stack web applications, AI backends, real-time streaming services, and background workers into secure, enterprise-compliant architectures.

---

## 📦 What's Included

```text
serverless-architect/
├── README.md                  # Compatibility & installation guide across AI harnesses
├── SKILL.md                   # Core advisory instruction set & diagnostic interview protocol
├── tests/
│   └── evals.json             # Comprehensive test suite (positive, negative, adversarial)
└── resources/
    └── decision_tree.md       # Interactive Mermaid decision trees & verified limit tables
```

---

## 🚀 Harness Compatibility & Installation

This skill adheres to the open markdown-based agent skill standard and works out-of-the-box with major AI coding assistants and harnesses:

### 1. Google Antigravity / Gemini CLI
Copy to your local or global skills directory:
```bash
mkdir -p ~/.antigravity/skills
cp -r serverless-architect ~/.antigravity/skills/
```

### 2. Claude Code
Copy the `serverless-architect` folder into your project or user skills directory:
```bash
mkdir -p .claude/skills
cp -r serverless-architect .claude/skills/
```

### 3. Cursor IDE
Link or copy into Cursor's rule directory:
```bash
mkdir -p .cursor/rules
cp serverless-architect/SKILL.md .cursor/rules/serverless-architect.mdc
```

### 4. Windsurf / Cascade
Add as a global or workspace rule:
```bash
cp serverless-architect/SKILL.md .windsurfrules
```

### 5. Roo Code / Cline
Add as a custom mode or append to `.clinerules` / `.roomodes`:
```bash
cat serverless-architect/SKILL.md >> .clinerules
```

### 6. Custom AI Agents / LangChain / CrewAI / ChatGPT
Inject `SKILL.md` and `resources/decision_tree.md` directly into your system prompt or knowledge base context.

---

## 🎯 What the Skill Does (Advisory Role)

1. **Conducts a Non-Technical Diagnostic Interview**: Asks plain-English, outcome-oriented questions (1–2 at a time) to evaluate cloud footprint, audience, protocol requirements, data topology, and throughput.
2. **Evaluates Enterprise Guardrails**: Enforces Zero-Trust SSO (GCP ALB + Serverless NEG + IAP / AWS ALB + OIDC), Direct VPC Egress for private databases, Secrets Management, and CMEK encryption.
3. **Applies Multi-Cloud Decision Logic**: Evaluates AWS (ECS Fargate, Lambda with Web Adapter, API Gateway WebSockets) vs. GCP (Cloud Run, Cloud Run Jobs, Cloud Run GPU).
4. **Highlights Scalability Bottlenecks & FinOps Traps**: Audits hard limits, regional concurrency caps, NAT Gateway bandwidth costs ($0.045/GB), RDBMS connection limits, and downstream SaaS API rate limits.
5. **Delivers Enterprise Architecture Specifications**: Outlines executive verdicts, security boundaries, compute sizing, runtime flags (`cpu-throttling`, Startup CPU Boost), and multi-cloud equivalency mapping.
