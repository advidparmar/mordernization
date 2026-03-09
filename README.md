# COBOL Modernization Prompt Library

A structured set of LLM prompts for converting a COBOL codebase into modernization-ready specifications. Designed to work with any coding agent (Claude Code, Cursor, Aider, Copilot, Windsurf, etc.) against an existing Git repository.

## Quick Start

1. Clone this prompt library into your COBOL repository (or keep it alongside)
2. Open `00_orchestrator.md` and update the configuration variables
3. Feed the orchestrator prompt to your coding agent
4. The agent will scan your repo, then work through each pass sequentially

## File Structure

```
cobol-modernization-prompts/
├── README.md                          ← You are here
├── 00_orchestrator.md                 ← START HERE: Master process that drives everything
├── 01_discovery_inventory.md          ← Pass 1: Inventory, dependencies, data dictionary
├── 02_business_specifications.md      ← Pass 2: Business processes, rules, workflows
├── 03_functional_specifications.md    ← Pass 3: Functional specs, API contracts, interfaces
├── 04_technical_specifications.md     ← Pass 4: Tech specs, Gherkin tests, migration maps
└── 05_governance_crosscutting.md      ← Pass 5: Traceability, risks, executive summary
```

## How It Works

### The Orchestrator (`00_orchestrator.md`)

The orchestrator is the entry point. It:
1. **Scans your repo** — finds all COBOL programs, copybooks, JCL, and BMS maps
2. **Builds a dependency map** — resolves COPY statements, traces CALL graphs
3. **Classifies programs into tiers** — Critical / Important / Commodity
4. **Identifies domain clusters** — groups related programs together
5. **Creates the output directory structure** — organized for both human and machine consumption
6. **Drives each pass** — referencing the individual phase files

### The Five Passes

Each pass has its own markdown file containing detailed prompts with:
- **Role definition** — what perspective the LLM should take
- **Input specification** — what files to read from the repo and prior outputs
- **Output templates** — exact structure for both human-readable and machine-readable artifacts
- **Quality gates** — what to verify before proceeding to the next pass

| Pass | File | What It Produces |
|------|------|-----------------|
| 1 | `01_discovery_inventory.md` | Inventory entries, dependency graphs, data dictionaries, domain model schemas (JSON Schema) |
| 2 | `02_business_specifications.md` | Business process narratives, workflow state machines (YAML), business rules with test cases (YAML DSL) |
| 3 | `03_functional_specifications.md` | Functional specs, API contracts (OpenAPI 3.1), interface specs, event specs (AsyncAPI) |
| 4 | `04_technical_specifications.md` | Technical specs with Java pseudocode, Gherkin test features, data migration mappings |
| 5 | `05_governance_crosscutting.md` | Traceability matrix, risk register, executive summary, completion report |

### Dual-Format Output

Every prompt produces **two outputs** from a single analysis:
- **Human-readable** (Markdown) — for business stakeholders, architects, auditors
- **Machine-readable** (YAML/OpenAPI/Gherkin) — for code generators, test runners, CI/CD

### Output Directory Structure

```
modernization-output/
├── 00_repo_scan/           ← Repo structure, dependencies, program tiers
├── 01_discovery/
│   ├── prose/              ← Human-readable inventory & data dictionaries
│   └── specs/
│       ├── inventory/      ← Program inventory YAMLs
│       └── domain-model/   ← JSON Schema entity definitions
├── 02_business/
│   ├── prose/              ← Business process documents
│   └── specs/
│       ├── workflows/      ← State machine YAMLs
│       └── rules/          ← Business rules with test cases
├── 03_functional/
│   ├── prose/              ← Functional specifications
│   └── specs/
│       ├── api-specs/      ← OpenAPI 3.1 service contracts
│       ├── interfaces/     ← File/MQ interface specs
│       └── events/         ← AsyncAPI event specs
├── 04_technical/
│   ├── prose/              ← Technical specs, data model, service decomposition
│   └── specs/
│       ├── features/       ← Gherkin/Cucumber test features
│       └── migration/      ← Data migration mapping YAMLs
├── 05_governance/
│   ├── prose/              ← Executive summary, risk narrative, traceability report
│   └── specs/              ← Traceability matrix YAML, risk register YAML
├── logs/
│   └── processing_log.yaml ← Progress tracking (supports resume)
└── COMPLETION_REPORT.md    ← Final summary with statistics and next steps
```

## Usage with Different Agents

### Claude Code
```bash
cd /path/to/cobol-repo
claude
# Paste the orchestrator prompt with updated variables
```

### Cursor / Windsurf
Open the COBOL repo in the IDE. Reference the orchestrator in the AI chat. The agent can read/write files directly.

### API Pipeline
Use the orchestrator logic as a script driver. For each program, assemble the prompt from the appropriate phase file, inject source code, call the API, and store output.

## Resuming After Interruption

The processing log tracks which programs are done. To resume:

```
Resume the COBOL modernization analysis.
Repository: /path/to/repo
Output: /path/to/output
Read the processing log at [output]/logs/processing_log.yaml 
and continue from where we left off.
Phase instructions are at: /path/to/cobol-modernization-prompts/
```

## Compatibility

These prompts work with any LLM that can:
- Read files from a local filesystem (or accept pasted code)
- Follow structured output templates
- Produce valid YAML and Gherkin syntax

Tested with: Claude (API/Chat/Code), GPT-4/o1, Gemini, GitHub Copilot, Cursor, Windsurf, Aider.
