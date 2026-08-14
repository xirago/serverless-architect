# Serverless Architect: Universal Agent Skill

A vendor-agnostic, enterprise-grade agent skill for diagnosing, selecting, and architecting serverless hosting platforms across **Amazon Web Services (AWS)** and **Google Cloud Platform (GCP)**.

Designed to turn "vibe-coded" prototypes, full-stack applications, AI backends, and background workers into secure, production-ready enterprise services.

---

## 📦 What's Included

```text
serverless-architect/
├── README.md                  # Compatibility & installation guide across AI harnesses
├── SKILL.md                   # Core instruction set & non-technical diagnostic protocol
└── resources/
    └── decision_tree.md       # Interactive Mermaid flowcharts & verified limit tables
```

---

## 🚀 Harness Compatibility & Installation

This skill adheres to the open markdown-based agent skill standard and works out-of-the-box with all major AI coding assistants and harnesses:

### 1. Claude Code
Copy the `serverless-architect` folder into your project or user skills directory:
```bash
mkdir -p .claude/skills
cp -r serverless-architect .claude/skills/
```

### 2. Cursor IDE
Link or copy into Cursor's rule directory:
```bash
mkdir -p .cursor/rules
cp serverless-architect/SKILL.md .cursor/rules/serverless-architect.mdc
```

### 3. Windsurf / Cascade
Add as a global or workspace rule:
```bash
cp serverless-architect/SKILL.md .windsurfrules
```

### 4. Roo Code / Cline
Add as a custom mode or append to `.clinerules` / `.roomodes`:
```bash
cat serverless-architect/SKILL.md >> .clinerules
```

### 5. Google Antigravity / Gemini CLI
Copy to your local or global skills directory:
```bash
mkdir -p ~/.antigravity/skills
cp -r serverless-architect ~/.antigravity/skills/
```

### 6. Custom AI Agents / LangChain / CrewAI / ChatGPT
Inject `SKILL.md` and `resources/decision_tree.md` directly into your system prompt or knowledge base context.

---

## 🎯 What the Skill Does

1. **Conducts a Non-Technical Diagnostic Interview**: Asks plain-English, outcome-oriented questions (1–2 at a time) to understand the app without overwhelming non-technical stakeholders.
2. **Evaluates Enterprise Guardrails**: Enforces Zero-Trust SSO (IAP / ALB OIDC), Direct VPC Egress for private databases, Secrets Management, and CMEK encryption.
3. **Applies Multi-Cloud Decision Logic**: Compares AWS (Lambda, App Runner, ECS Fargate) vs. GCP (Cloud Run, Cloud Run Jobs, Cloud Functions Gen 2).
4. **Highlights Scalability Bottlenecks**: Audits hard limits, regional concurrency caps, NAT port exhaustion, RDBMS connection limits, and downstream SaaS API rate limits.
5. **Produces Production Scaffolds**: Delivers ready-to-run `Dockerfile` or OpenTofu/Terraform infrastructure templates.
