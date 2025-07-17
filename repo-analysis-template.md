<task>
Your task is to act as an AI code analysis and refactoring advisor. You will perform a deep analysis of the repository to create a **correctly rendered, visually organized knowledge graph** that explicitly highlights component relationships and change impact, along with a detailed analysis report.

**Core Principle:** Your absolute highest priority is to generate a final `knowledge_graph.html` where the Mermaid diagram renders successfully. You must achieve this by following the syntax and template rules below with extreme precision. **Under no circumstances should you write or execute helper scripts.**

---

### **Critical Rules for Generating a Successful Report**

**1. Mermaid Syntax Rules:**
*   **Safe IDs:** All node and subgraph IDs must be a single string containing only letters, numbers, and underscores (`_`). Replace all other characters (`/`, `.`, `-`, `.`) with an underscore.
*   **Quote All Labels:** All human-readable text for nodes and subgraphs **must** be enclosed in double quotes.
*   **Separate Definitions from Relationships:** Define ALL subgraphs and nodes first. List ALL relationship links (`-->`) at the end of the Mermaid block.
*   **Use CSS Classes for Styling:** Add a CSS class to your node definitions for styling. Example: `my_function("my_function()"):::functionNode`.
*   **Descriptive Relationship Labels:** Use concise and descriptive labels for relationships, clearly indicating the nature of the interaction (e.g., `imports`, `calls`, `inherits`, `modifies data in`, `accesses`).
*   **Impact Highlighting in Graph:** For nodes identified as "High Impact" (see Phase 3), consider using distinct styling (e.g., a bolder border, a specific fill color) via a dedicated CSS class (e.g., `highImpactNode`).

**2. HTML Template Rule:**
*   You **must** use the exact HTML template provided at the end of this prompt. It contains the modern, correct JavaScript imports and configurations required for Mermaid v10+ to render properly. **Do not modify the template's script or style sections.**

---

### **Phase 1: File Discovery**

**Goal:** To get a clean, filtered list of relevant, user-written source files, ignoring third-party and minified code, across multiple common programming languages.

**Execution Steps:**
1.  **Get Full List:** Run `git ls-files --cached --others --exclude-standard`.
2.  **Apply Filters:** Create a new list, removing files from:
    *   `lib/`, `vendor/`, `node_modules/`, `dist/`, `build/`, `.git/`, `__pycache__/`, `target/` (for Java/Rust), `bin/`
    *   Patterns like `*.min.*`, `*.test.*`, `*.spec.*`, `*.d.ts`, `*_generated.*`
    *   Common binary or documentation files: `*.jar`, `*.exe`, `*.pdf`, `*.md`, `*.txt`, `LICENSE`
    Prioritize main source code files.

---

### **Phase 2: Code Inventory Creation (Deepened for Granular Relational Data)**

**Goal:** To perform a full analysis on the filtered files and create a detailed JSON inventory that explicitly captures granular relational information and potential interactions.

**Execution Steps:**
1.  **Create Directory:** Ensure `./codeanalysis/` exists.
2.  **Analyze Filtered Files:** For each file in your filtered list, perform an in-depth static analysis to identify:
    *   **`file_path`**: The full path of the file.
    *   **`language`**: Detected programming language (e.g., `python`, `javascript`, `java`, `typescript`, `go`, etc.).
    *   **`definitions`**: An array of identified top-level structural constructs within the file. For each:
        *   `name`: The name (e.g., `main`, `MyClass`, `process_data`).
        *   `type`: (e.g., `function`, `method`, `class`, `interface`, `variable`, `constant`, `enum`).
        *   `starts_line`: Line number where definition begins.
        *   `ends_line`: Line number where definition ends.
    *   **`imports_dependencies`**: An array of explicit module/file dependencies declared. For each:
        *   `target_name`: Name of the imported module/file (e.g., `os`, `requests`, `../utils/helpers.py`).
        *   `type`: (e.g., `internal_module`, `external_library`, `local_file`).
    *   **`call_references`**: An array of detected calls from this file or its definitions to other functions/methods. For each:
        *   `source_definition_name`: The name of the function/method making the call (or `_file_level` if not within a definition).
        *   `target_name`: The name of the function/method being called (e.g., `read_prompt_from_file`, `obj.method_name`).
        *   `call_type`: (e.g., `direct_function_call`, `method_call`, `static_method_call`, `constructor_call`).
        *   `line_number`: Line number where the call occurs.
    *   **`inheritance_relations`**: An array of detected inheritance relationships (for class-based languages). For each:
        *   `child_class_name`: The name of the inheriting class.
        *   `parent_class_name`: The name of the class being inherited from.
    *   **`data_structure_definitions`**: An array of complex data structure definitions (e.g., structs, dataclasses, interfaces, Pydantic models). For each:
        *   `name`: Name of the data structure.
        *   `fields`: A simplified list of its primary fields.
    *   **`side_effects_potential`**: An array of detected operations that could have side effects beyond the function's return value. For each:
        *   `type`: (e.g., `file_io_write`, `database_write`, `network_request_outgoing`, `global_variable_modification`, `environment_variable_read`).
        *   `target_info`: Specific file path, DB table, API endpoint, etc., if discernible.
        *   `source_definition_name`: Function/method where the side effect is initiated.
3.  **Create JSON:** Structure a comprehensive JSON object with `metadata` (including `generated_at` timestamp and `source`) and a `files` section, keyed by `file_path`.
4.  **Save Inventory:** Save the object to a timestamped file: `./codeanalysis/YYYY-MM-DD_HH-MM-SS_code_inventory.json`.

---

### **Phase 3: Structured Graph and Report Generation (Deepest Relational Analysis and Impact Assessment)**

**Goal:** To create a visually organized graph that explicitly models dependencies and potential impact, and a detailed report providing actionable insights for development and refactoring.

**Execution Steps:**
1.  **Load Inventory:** Load the most recent inventory JSON file.
2.  **Perform Deep Relational and Impact Analysis:**
    *   **Resolve All Dependencies:** Trace all `imports_dependencies` and `call_references` to their ultimate definitions *across the entire codebase*. Build a complete Directed Acyclic Graph (DAG) for dependencies where possible.
    *   **Calculate Component "Impact Score" (Fan-in/Indegree):** For every file, class, and significant function/method:
        *   Count the number of distinct components (files, classes, functions) that *call* it, *import* it, or *inherit from* it.
        *   Classify impact: `High Impact` (top 10% or custom threshold), `Medium Impact`, `Low Impact`.
    *   **Identify Critical Execution Paths/Workflows:** Trace 1-3 primary execution flows (e.g., from `main` or main API entry points) to highlight how key components interact in a sequence.
    *   **Detect Orphaned Entities/Dead Code Candidates:** Identify functions, classes, or files with an "Impact Score" of 0 (no incoming calls, imports, or inheritance) after thorough analysis.
    *   **Detect Circular Dependencies:** Clearly identify instances where A depends on B, and B depends on A (or longer cycles).
    *   **Stale/Infrequently Modified Files:** Use `git log --pretty=format:%ad --date=iso-strict -- [file_path]` to get the last commit date for each relevant file. Flag files not modified in the last 12-24 months.
    *   **Identify Ownership/Last Contributor:** For stale or high-impact files, try to identify the last active contributor using `git log -1 --pretty=format:%an -- [file_path]`. (If direct git access is limited, infer or state this limitation).
    *   **Refactoring Opportunities (Specific & Actionable):**
        *   **High Coupling:** List components with exceptionally high incoming OR outgoing dependencies that suggest tight coupling needing decoupling.
        *   **God Objects/Functions:** Identify single entities with an excessive number of responsibilities or a very broad range of `call_references` and `side_effects_potential`.
        *   **Redundancy:** Highlight similar `definitions` or `call_references` in different files suggesting code duplication.
        *   **Modularity Suggestions:** Propose how highly coupled components might be broken down or regrouped into more cohesive modules.
    *   **Potential Security & Performance Hotspots:** Based on `side_effects_potential` (especially network/DB I/O, file writes, env var access), flag areas requiring security review or performance optimization.
3.  **Build the Mermaid Graph (Explicit Impact Visualization):**
    *   Start your Mermaid list with `graph TD;`.
    *   **Prioritize Logical Groupings (Subgraphs):**
        *   `subgraph Core Engine`
        *   `subgraph Agents Layer`
        *   `subgraph Utilities & Helpers`
        *   `subgraph Deprecated Code`
        *   `subgraph Testing Framework`
        *   `subgraph Configuration & Data Models`
    *   **Define All Nodes First:** Iterate through your inventory. Create nodes for:
        *   **Files:** `file_path_safe_id("filename.py"):::fileNode`
        *   **Classes:** `class_name_safe_id("ClassName"):::classNode`
        *   **Key Functions:** `function_name_safe_id("function_name()"):::functionNode` (focus on public or frequently called functions, not every private helper).
        *   **Data Structures:** `data_struct_id("DataStructure"):::dataNode`
        *   Apply `highImpactNode` class to nodes identified as such.
        *   Apply `deprecatedNode` class to nodes/files within `_depricated` folders or explicitly marked as such.
    *   **Define All Relationships (Post-Definition):** After all nodes are defined, list all relationship lines. Use explicit labels.
        *   `source_id -->|imports| target_id`
        *   `caller_func_id -->|calls| called_func_id`
        *   `child_class_id -->|inherits| parent_class_id`
        *   `func_id -->|modifies_data_in| file_or_db_id` (If `side_effects_potential` targets an entity represented in the graph)
        *   `func_id -->|accesses_env| env_node_id` (If useful to represent env interactions)
        *   `func_id -->|calls_api| external_service_node_id`
    *   **Strategic Node Exclusion:** For very large files, create nodes only for the file itself and its most critical/public functions to prevent graph overload.
4.  **Generate Sidebar Report (HTML with Detailed Analysis & Actionable Insights):** Create structured HTML for the sidebar containing your analysis. This should explicitly support decision-making for changes.

    *   **Overall Codebase Summary:**
        *   Total Files Analyzed, Total Lines of Code (estimate or state if not measured directly).
        *   Primary Languages Detected.
        *   Timestamp of Analysis.
    *   **Component Impact & Criticality:**
        *   <h3>High Impact Components (Changes Here Ripple Widely)</h3>
            *   List Top N files/classes/functions by impact score, along with their score and a brief explanation of *why* they are high impact (e.g., "depended on by 15 modules, core configuration").
        *   <h3>Key Workflows & Data Flows</h3>
            *   Brief descriptions of 1-3 core application workflows (e.g., "User Request -> Main Loop -> Agent Orchestration -> Report Generation").
            *   Highlight critical components in each flow.
    *   **Code Health & Opportunities:**
        *   <h3>Orphaned Entities & Dead Code Candidates</h3>
            *   List files/functions/classes that appear unused, with their inferred location/context.
            *   **Action:** "Review for removal or identify missing references."
        *   <h3>Stale & Dormant Files</h3>
            *   List files last modified > 12-24 months ago. Include last contributor if available.
            *   **Action:** "Review for relevance, update, or archive. Consult last contributor."
        *   <h3>Circular Dependencies Detected</h3>
            *   List specific cycles (e.g., `A.py <-> B.py`).
            *   **Action:** "Refactor to break the cycle, often by introducing an intermediary module or re-evaluating responsibilities."
        *   <h3>High Coupling & Refactoring Targets</h3>
            *   List specific files/classes/functions with very high fan-in/fan-out.
            *   **Action:** "Consider splitting these into smaller, more focused units ("Single Responsibility Principle")."
        *   <h3>Potential Duplication</h3>
            *   Point out any patterns suggesting similar code exists in multiple places.
            *   **Action:** "Consolidate into shared utilities or base classes."
    *   **Risk Assessment (Inferred):**
        *   <h3>Security/Performance Hotspots</h3>
            *   List files/functions identified with significant `side_effects_potential` (e.g., frequent DB writes, external API calls, complex file I/O).
            *   **Action:** "Prioritize code review, performance profiling, and error handling in these areas."
    *   **Impact Prediction Guidance:**
        *   **Hypothetical Change Example:** "If you were to change `deepoutputs_engine/config.py:resolve_model_env` (High Impact), the following components would likely require review or modification:"
        *   List 3-5 of the most direct dependents identified from your impact score calculation.
        *   **General Advice:** "Changes to components listed under 'High Impact Components' in this report require thorough testing and careful consideration of all transitive dependencies."

---

### **Phase 4: Final HTML Report Generation**

**Goal:** To create the final report using the **correct and modern** HTML template provided below, ensuring all dynamic content is correctly embedded.

**Execution Steps:**
1.  **Consolidate and Embed:** Place your final Mermaid syntax and sidebar report HTML strings into the robust HTML template below.
2.  **Save File:** Save the complete string as `knowledge_graph.html`.

---

### **Final HTML Output Template (Modern and Correct)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Codebase Analysis Report</title>
    <style>
        body { margin: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol"; display: flex; flex-direction: column; align-items: center; background-color: #f0f2f5; color: #34495e; line-height: 1.6; }
        h1, h2, h3 { color: #2c3e50; margin-top: 1.5em; margin-bottom: 0.5em; border-bottom: 1px solid #ebf0f4; padding-bottom: 0.3em; }
        p { margin-bottom: 1em; }

        #report-header { width: 98%; text-align: center; padding: 20px 0; background-color: #ffffff; border-bottom: 1px solid #e0e6ea; box-shadow: 0 2px 4px rgba(0,0,0,0.05); border-radius: 8px; margin-top: 20px;}
        #report-container { display: flex; width: 98%; gap: 20px; margin-top: 20px; align-items: flex-start; }
        
        #graph-container { flex-grow: 1; border: 1px solid #e0e6ea; padding: 15px; border-radius: 8px; background-color: #ffffff; box-shadow: 0 4px 12px rgba(0,0,0,0.08); min-height: 70vh; display: flex; flex-direction: column; }
        #graph-container h2 { text-align: center; }
        .mermaid { flex-grow: 1; /* Allow Mermaid to take available space */ }

        #sidebar { width: 400px; flex-shrink: 0; border: 1px solid #e0e6ea; padding: 15px; border-radius: 8px; max-height: 85vh; overflow-y: auto; background-color: #ffffff; box-shadow: 0 4px 12px rgba(0,0,0,0.08); }
        #sidebar h2 { margin-bottom: 1em; text-align: center; }
        #analysis-report ul { list-style-type: none; padding-left: 0; }
        #analysis-report li { background-color: #f9fbfb; border: 1px solid #eef3f7; padding: 10px; margin-bottom: 8px; border-radius: 6px; font-size: 0.95em; }
        #analysis-report li strong { color: #d64545; /* Critical */ }
        .info { color: #3498db; }
        .warning { color: #f39c12; /* Orange for warnings */ }
        .critical { color: #d64545; /* Red for critical issues */ }
        .success { color: #27ae60; /* Green for positive notes */ }

        /* Mermaid Node Styling - Extensive and Intentional */
        .fileNode { fill:#E0F2F7; stroke:#A7D9EE; stroke-width:2px; } /* Light blue, for files */
        .classNode { fill:#DDA0DD; stroke:#9932CC; stroke-width:2.5px; font-weight: bold; } /* Medium Purple, bold for classes */
        .functionNode { fill:#C9EDF8; stroke:#48A0D0; stroke-width:2px; } /* Slightly darker blue, for functions */
        .dataNode { fill:#FFFACD; stroke:#FFD700; stroke-width:2px; } /* Light yellow, for data structures */
        .importedLibNode { fill:#e1ffe1; stroke:#66BB6A; stroke-width:2px; } /* Light green, for external libraries */
        .apiServiceNode { fill:#FFDDDD; stroke:#FF3D57; stroke-width:2px; } /* Light red, for external API services */
        .environmentNode { fill:#D1E7DD; stroke:#1C8B49; stroke-width:2px; } /* Darker green, for environment variables */

        /* Impact-based Styling */
        .highImpactNode { fill:#F5E0E3; stroke:#E21A3B; stroke-width:3px; border-style: solid; } /* Pale red with strong red border */
        .deprecatedNode { fill:#F0F0F0; stroke:#C0C0C0; stroke-width:1.5px; border-style: dashed; } /* Greyed out, dashed for deprecated */
        .orphanNode { fill:#F2F2F2; stroke:#A0A0A0; stroke-width:1.5px; border-style: dotted; } /* Dotted border for orphans */
        
        /* Specific Mermaid Link Styles for better visual differentiation on types of relationships */
        .link[id*="imports"] path { stroke: #2ecc71; stroke-width: 1.5px; } /* Green for imports */
        .link[id*="calls"] path { stroke: #3498db; stroke-width: 1.5px; } /* Blue for calls */
        .link[id*="inherits"] path { stroke: #9b59b6; stroke-width: 1.5px; stroke-dasharray: 5 5; } /* Purple dashed for inheritance */
        .link[id*="modifies_data_in"] path, .link[id*="accesses"] path { stroke: #e74c3c; stroke-width: 2px; } /* Red for critical data/resource interaction */
        .link[id*="calls_api"] path { stroke: #f39c12; stroke-width: 1.5px; stroke-dasharray: 2 2;} /* Orange dashed for API calls */

        /* Styling for Mermaid labels (text over arrows) */
        .edgeLabel { font-size: 0.8em; background-color: #ffffff; padding: 2px 5px; border: 1px solid #e0e6ea; border-radius: 3px; }

    </style>
</head>
<body>
    <div id="report-header">
        <h1>Codebase Analysis Report</h1>
        <p>A comprehensive overview of repository structure, dependencies, and potential impact areas for effective code changes.</p>
    </div>
    <div id="report-container">
        <div id="graph-container">
            <h2>Knowledge Graph</h2>
            <pre class="mermaid">
%% --- PASTE YOUR GENERATED MERMAID SYNTAX HERE --- %%
            </pre>
        </div>
        <div id="sidebar">
            <h2>Analysis & Refactoring Insights</h2>
            <div id="analysis-report">
                <!-- %% --- PASTE YOUR GENERATED SIDEBAR REPORT HTML HERE --- %% -->
            </div>
        </div>
    </div>
    <script type="module">
        import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
        
        mermaid.initialize({
            startOnLoad: true,
            theme: 'default', // Consider 'neutral', 'forest', 'dark' for different aesthetics
            fontFamily: 'inherit', // Use system font
            mermaid: {
              curve: 'catmullRom', // A smoother curve for better readability
              htmlLabels: true
            },
            flowchart: {
                htmlLabels: true,
                curve: 'catmullRom'
            },
            sequence: {
                diagramMarginX: 50,
                diagramMarginY: 10,
                boxMargin: 10,
                actorMargin: 50,
                width: 150,
                height: 65,
                boxTextMargin: 5,
                lineMargin: 10,
                todayLineColor: "#FF0000",
                arrowMainGap: 10,
                labelBoxWidth: 100,
                labelBoxHeight: 30,
                sectionMargin: 10,
                exectionLineHeight: 30,
                exectionBorderAndTextMargin: 5,
                messageAlign: "center",
                mirrorActors: true,
                forceKeys: false,
                wrapTexts: true,
                loopAltBackground: "#92D6EB",
                noteBkgColor: "#FFF5AD",
                noteTextColor: "#333",
                activationBkgColor: "#F47979",
                markerFontColor: "#fff",
                transitionTime: 500 // Animation speed for new elements.
             },
            git: {
                rotate: 0,
                fontSize: 12,
                labelHeight: 40,
                nodeSpacing: 60,
                nodeRadius: 10,
                tagSpacing: 2
            }
        });
    </script>
</body>
</html>
```
</task>