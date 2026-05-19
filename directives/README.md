# Layer 1 Directive: Build & Orchestrate Transcript Intelligence

You are an expert AI developer agent operating in Layer 2 (Orchestration). Your objective is to build, configure, and deploy **Transcript Intelligence**—a production-ready, cost-aware B2B Enterprise SaaS call transcript analytics pipeline utilizing a **3-Layer Agentic Architecture**.

Follow these comprehensive, natural-language instructions to implement the system from scratch.

---

## 🎯 High-Level Goal
Build a pipeline that ingests meeting transcripts recursively, runs them through three modular LLM analysis stages (Categorizer, Sentiment, and Insight Generator), caches outputs to prevent double-billing of tokens, and automates cloud execution and result commit-backs via a GitOps workflow.

---

## 👥 Role Assignment & Separation of Concerns

Ensure your implementation strictly adheres to the following separation of duties:
* **Layer 1 (The Directive Author - Expert Prompts):** Create and store system prompts in modular markdown files inside [`SKILLS/`](file:///Users/abhi/Documents/github/transcript-intelligence/SKILLS/).
* **Layer 2 (The Orchestrator - Workflow Trigger):** Configure GitHub Actions (`.github/workflows/process-transcripts.yml`) to schedule runs, manage action runners' cache, and securely push processed data back.
* **Layer 3 (The Executor - Deterministic Script):** Implement a Python pipeline in [`execution/process_transcripts.py`](file:///Users/abhi/Documents/github/transcript-intelligence/execution/process_transcripts.py) to manage I/O, dataset parsing, and cache checks.

---

## 🛠️ Step-by-Step Implementation Instructions

### Phase 1: Environment & Directory Setup
1. **Initialize Workspace:** Establish a standard workspace layout containing a root `README.md`, `datasets/`, `SKILLS/`, `execution/`, and `results/`.
2. **Set Dependencies:** Create `execution/requirements.txt` containing `pandas`, `tqdm`, `openai`, `google-generativeai`, and `python-dotenv`.
3. **Configure API Lookup:** Program the system to dynamically check the environment for a `GITHUB_TOKEN` (for Azure OpenAI Models at `https://models.inference.ai.azure.com`) or `GEMINI_API_KEY` (for Google Gemini). If neither is configured, fallback safely to local Ollama (`localhost:11434` with model `llama3`) or structured Mock outputs to prevent program crashes.
4. **Input Schema Specifications:**
   - **`meeting-info.json`**: Must be parsed to extract metadata fields:
     ```json
     { "meetingId": "string", "title": "string", "host": "string", "duration": number }
     ```
   - **`transcript.json`**: Contains dialogue sentence objects. Must be chronologically reconstructed into speaker turns:
     ```json
     { "data": [ { "speaker_name": "string", "sentence": "string" } ] }
     ```

### Phase 2: Create Modular Skills & Enforce Response Schemas (Layer 1 Directives)
Develop three distinct directories inside `SKILLS/` to isolate prompt personas. Each skill must contain a `SKILL.md` file starting with a **YAML frontmatter header** to allow custom hyper-parameter overrides without changing Python code:

1. **`meeting-categorizer`** ([`SKILLS/meeting-categorizer/SKILL.md`](file:///Users/abhi/Documents/github/transcript-intelligence/SKILLS/meeting-categorizer/SKILL.md)):
   - *Parameters:* `temperature: 0.0`, `topP: 0.1`, `maxTokens: 1000`.
   - *Persona:* Aegis Cloud system classifier.
   - *Constraints:* Enforce domains; if all emails end with `@aegiscloud.com` or `@aegis.com`, classify as **Internal**. Extract 2 to 4 major themes.
   - *Expected JSON Output:*
     ```json
     { "category": "Customer Support | External Customer | Internal", "themes": ["theme1", "theme2"] }
     ```
2. **`sentiment-analyzer`** ([`SKILLS/sentiment-analyzer/SKILL.md`](file:///Users/abhi/Documents/github/transcript-intelligence/SKILLS/sentiment-analyzer/SKILL.md)):
   - *Parameters:* `temperature: 0.1`, `topP: 0.25`, `maxTokens: 800`.
   - *Persona:* Enterprise SaaS sentiment analyst.
   - *Constraints:* Score tone strictly from `-1.0` (highly negative) to `+1.0` (highly positive) and write a 2-3 sentence narrative tracing the emotional arc trajectory.
   - *Expected JSON Output:*
     ```json
     { "score": 0.35, "trajectory": "Speaker expressed initial concern but resolved with positive customer feedback." }
     ```
3. **`insight-generator`** ([`SKILLS/insight-generator/SKILL.md`](file:///Users/abhi/Documents/github/transcript-intelligence/SKILLS/insight-generator/SKILL.md)):
   - *Parameters:* `temperature: 0.7`, `topP: 0.9`, `maxTokens: 1500`.
   - *Persona:* Elite SaaS product advisor.
   - *Constraints:* Brainstorm **EXACTLY TWO** customized, non-obvious, actionable roadmap recommendations specific to Aegis Cloud's product modules.
   - *Expected JSON Output:*
     ```json
     { "strategic_insights": ["Insight recommendation 1 details...", "Insight recommendation 2 details..."] }
     ```

### Phase 3: Create the Deterministic Python Ingestion & Execution Engine (Layer 3)
Build the core execution pipeline in `execution/process_transcripts.py`:
1. **Dynamic Path Resolution:** Program the script to resolve all project roots dynamically relative to its absolute file path, guaranteeing seamless execution on any machine or runner.
2. **Frontmatter Config Parser (`parse_skill`):** Write a dynamic parser to extract YAML frontmatter parameters (`temperature`, `topP`, `maxTokens`) and pass them to the LLM client call.
3. **Recursive Ingestion Loader:** Implement recursive traversal of the `datasets/` folder to discover meetings under flat layouts or date-based subdirectories (e.g. `datasets/YYYYMMDD/meeting_id`).
4. **Persistent Caching System:**
   - Establish a caching layer targeting `.tmp/cache/`.
   - Before hitting the LLM API, compute a unique key from `meetingId`. If a cache file exists, parse and return it instantly to protect compute and rate limits.
5. **Flatten, Mirror & Export:** Compile all processed meetings into a single DataFrame using Pandas, and export day-specific databases (`analysis_results.csv` and `analysis_results.json`) strictly inside their respective date subfolders (e.g. `results/YYYYMMDD/analysis_results.csv` and `results/YYYYMMDD/analysis_results.json`), together with each individual meeting's stage-by-stage JSON analysis report inside a mirrored subfolder under `results/` matching the raw `datasets/` input path structure (e.g. `results/YYYYMMDD/meeting_id/analysis_results.json`).
6. **Macro Trends Reporter:** Program the pipeline to compute high-level aggregate trends (average sentiment, counts by category, durations) for each date folder, writing a segment-aware C-Suite Executive Insights slide report (`insights_summary.md`) strictly inside each respective date folder (e.g. `results/YYYYMMDD/insights_summary.md`). Ensure no global databases or macro summaries clutter the root of the results directory.
7. **Decoupled UI Presentation Layer:** Compile the entire daily macro summary into a pure `data.js` script asset, and output it exclusively into a segregated `ui/` directory alongside the dashboard HTML file, bypassing CORS blockades entirely and ensuring no UI assets clutter the analytical `results/` outputs.

### Phase 4: Create the Automated GitOps Pipeline (Layer 2 Orchestration)
Build the CI/CD orchestrator file at `.github/workflows/process-transcripts.yml`:
1. **Define Triggers:** Configure the pipeline to run on pushes affecting files under `datasets/**`.
2. **Implement Runner Caching:** Configure GitHub's broad cache system fallback using `actions/cache` to restore and save `.tmp/cache` across runs, ensuring zero redundant API fees.
3. **Secure Auto-Commit Push:** Set GITHUB_TOKEN permissions to write, staging and committing updated `results/` and `ui/` directories straight back into the main repository branch.

---

## 🛡️ Edge Cases & Error Handling Constraints

* **Corrupt JSON Ingestion:** Program Stage processors with robust try-catch blocks. If the LLM generates unparsable JSON syntax, assign a default structured fallback dictionary instead of crashing the thread.
* **Missing JSON Keys:** Ensure missing properties (`meeting-info.json` or `transcript.json`) fail gracefully, skipping that meeting while continuing to process the remaining dataset folders.
* **Token Rate Limits:** LOCAL cache checks must run in constant time ($O(1)$) to bypass LLM query limits entirely during duplicate pipeline executions.