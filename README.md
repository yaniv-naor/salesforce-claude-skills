# Salesforce Skills for Claude Code – AI-Powered Metadata Generation

An intelligent collection of specialized skills for **Claude Code** that automate the creation of Salesforce metadata components. Designed to work seamlessly with Claude Code's skill system, these skills enable developers to build complete Salesforce projects faster by generating objects, fields, triggers, flows, validation rules, permissions, and more — all with proper naming conventions, validation, and best practices built-in.

**Compatible with Claude Code CLI, Desktop App, Web App (claude.ai/code), and IDE extensions (VS Code, JetBrains).**

![Salesforce Skills](https://img.shields.io/badge/Salesforce-DX-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-purple)

---

## Contributors

- [Sarit Katz](https://github.com/sarit-katz)
- [Yaniv Naor](https://github.com/yaniv-naor)

---

## Demo

<img width="1280" height="766" alt="Claude Code Salesforce Skills" src="https://github.com/user-attachments/assets/88858744-0adb-4cc4-a205-23336ed3e5ff" />

---
## Features

- **Object Creation** – Automatically generate custom objects with proper sharing models, name fields, and deployment configuration.
- **Field Generation** – Create all field types: Text, Number, Picklist, Lookup, Master-Detail, Formula, Rollup Summary, and more.
- **Apex Development** – Generate Apex classes, triggers, test classes, async handlers (Queueable/Batch), and invocable methods.
- **Flow Automation** – Build Record-Triggered, Autolaunched, and Screen Flows with proper configuration.
- **Lightning Web Components** – Create LWC components with HTML, JS, CSS, test files, and optional Apex controllers.
- **Validation Rules** – Generate validation rules with proper formula syntax and error handling.
- **Permission Sets** – Configure object/field permissions, Apex access, and tab visibility.
- **Translation Support** – Automatically translate metadata (objects, fields, picklists, tabs) to Hebrew and Arabic.
- **Page Layouts & Apps** – Create Lightning apps, custom tabs, page layouts, and compact layouts.
- **Sharing Rules** – Configure organization-wide defaults and sharing rules.
- **Project Utilities** – Auto-detect project prefix, validate metadata, and debug scratch org deployments.
- **Multi-Language Support** – Full Hebrew (RTL) and Arabic translation capabilities built-in.
- **Best Practices Enforcement** – Automatic naming conventions, validation, and Salesforce metadata standards.

---

# Installation

## 1. Clone the repository

```bash
git clone https://github.com/yaniv-naor/salesforce-claude-skills.git
cd salesforce-claude-skills
```

## 2. Copy skills to your Claude Code skills directory

### Windows

```bash
cp -r sf-* "C:\Users\<YourUsername>\.claude\skills\"
```

### macOS / Linux

```bash
cp -r sf-* ~/.claude/skills/
```

## 3. Verify installation

Open Claude Code and type `/` to verify that all `sf-*` skills appear in the available skills list.

> **Note:** You can also install individual skills by copying only the specific skill folders you need.

---

# Available Skills

## Core Metadata Generation

- `sf-object-create` – Create custom objects with sharing models, name fields, and features.
- `sf-field-create` – Create all field types (Text, Number, Lookup, Master-Detail, Formula, Rollup Summary, Picklist, etc.).
- `sf-validation-create` – Generate validation rules with proper formula syntax and error messages.

## Apex Development

- `sf-apex-create` – Generate Apex classes with test coverage (75%+).
- `sf-apex-async` – Create Queueable, Batch, and Schedulable Apex classes.
- `sf-apex-invocable` – Build invocable methods for Flow integration.
- `sf-trigger-create` – Generate triggers with handler pattern and test classes.

## UI & Automation

- `sf-lwc-create` – Create Lightning Web Components (HTML, JS, CSS, tests, Apex controller).
- `sf-flow-create` – Build Record-Triggered, Autolaunched, and Screen Flows.
- `sf-page-create` – Generate Lightning apps, custom tabs, page layouts, and compact layouts.

## Security & Permissions

- `sf-permission-create` – Configure permission sets with object/field/Apex/tab permissions.
- `sf-sharing-configure` – Set up organization-wide defaults and sharing rules.

## Localization

- `sf-translation-create` – Translate metadata to Hebrew and Arabic (objects, fields, picklists, tabs, apps).

## Project Utilities

- `sf-prefix-detect` – Auto-detect project naming prefix from existing metadata.
- `sf-validate-all` – Run dry-run validation on all metadata before deployment.
- `sf-scratch-debug` – Troubleshoot scratch org deployment errors.

---

# Usage Examples

## Automatic Skill Detection

Simply describe what you want to build in natural language:

```text
"Create a custom object to track customer invoices with auto-numbering"
"Add a Status picklist field to the Invoice object"
"Create a trigger that updates total amount when line items change"
"Translate all metadata to Hebrew"
```

Claude Code automatically detects and uses the appropriate skill ✅

---

## Manual Skill Invocation

You can also invoke skills manually using the `/` prefix:

```text
/sf-object-create
/sf-field-create
/sf-translation-create
```

---

## Building Custom Objects

1. Describe the object and fields you need.
2. Claude Code detects the project prefix automatically.
3. Objects and fields are generated with proper naming conventions.
4. Metadata is validated before deployment.

---

## Creating Relationships

- Request Master-Detail or Lookup relationships.
- Skills automatically configure sharing models.
- Rollup Summary fields are created when needed.
- Parent-child relationships are properly linked.

---

## Translating Metadata

- Request translation to Hebrew or Arabic.
- Metadata is automatically translated (objects, fields, picklists, tabs, apps).
- Translation files are generated in proper Salesforce format.
- RTL languages are fully supported.

---

## Complete Project Build

Describe an entire system (e.g., `"Build a Vehicle Management System"`).

Claude Code automatically orchestrates multiple skills:

- Objects
- Fields
- Relationships
- Flows
- Apex
- Permissions
- Layouts
- Translations

Everything is validated before deployment to a scratch org.

---

# Technologies & Tools

- **Claude Code** – AI-powered development assistant with skill system.
- **Salesforce Extension Pack (VS Code)** – Official Salesforce extension for VS Code.
- **Salesforce CLI (`sf`)** – Command-line interface for Salesforce operations.
- **Git** – Version control for Salesforce source code.
- **Dev Hub & Scratch Orgs** – Ephemeral Salesforce development environments.

---

# Development

## Project Structure

Each skill is contained in its own folder with a `skill.md` file:

```text
salesforce-claude-skills/
├── sf-object-create/
│   └── skill.md
├── sf-field-create/
│   └── skill.md
├── sf-apex-create/
│   └── skill.md
├── sf-trigger-create/
│   └── skill.md
├── sf-flow-create/
│   └── skill.md
├── sf-lwc-create/
│   └── skill.md
├── sf-validation-create/
│   └── skill.md
├── sf-permission-create/
│   └── skill.md
├── sf-translation-create/
│   └── skill.md
├── sf-page-create/
│   └── skill.md
├── sf-sharing-configure/
│   └── skill.md
├── sf-translation-create/
│   └── skill.md
├── sf-prefix-detect/
│   └── skill.md
├── sf-validate-all/
│   └── skill.md
└── sf-scratch-debug/
    └── skill.md
```

---

# Contributing

1. Fork the repository.
2. Create a new branch:

```bash
git checkout -b feature/new-skill
```

3. Add or modify skills using the `skill.md` format.
4. Test the skill with Claude Code.
5. Submit a Pull Request.

---

# Skill Development Guidelines

- Follow the existing `skill.md` format with frontmatter metadata.
- Include clear trigger keywords for automatic detection.
- Provide step-by-step instructions for Claude Code.
- Add examples and troubleshooting sections.
- Test with multiple Salesforce project types.

---

# Related Tools & Extensions

## RTL Support for Claude Code

Working with Hebrew and Arabic translations? Check out the RTL Text Direction Extension:

### Claude Code RTL Extension (VS Code Marketplace)

Features:

- Automatic text direction detection for Hebrew (`עברית`) and Arabic (`العربية`)
- Seamless integration with `sf-translation-create`
- Proper RTL rendering in Claude Code interfaces

Install via command:

```bash
code --install-extension yaniv-naor.claude-code-rtl-fix
```

Perfect companion for translating Salesforce metadata to RTL languages 🌍

---

# License

MIT License.

---

# Issues and Support

If you encounter issues or want to contribute, please open an Issue or Pull Request on GitHub.

When reporting issues, please include:

- Skill name (e.g., `sf-object-create`)
- Error message or unexpected behavior
- Salesforce CLI version:

```bash
sf --version
```

- Claude Code version
- Steps to reproduce
