## LangGraph

LangGraph is an automatic AI agent based on LLM, including **a total contral agent + multiple specialized research agents + clear intermediate products + manual checklist review**.  
- can serve as <font color=red> a hard constraint, rollback, phased research agent, suitable for partial fixed pipeline + partial exploratory research tasks</font>.
- Rather than only one agent directly read and rewrite the whole paper, such as:bolt (2024), **windsurf** (2025.3), **cursor** (2025), **claude code** (2026.2), **codex** (2026.3).
- Rather than **AgentGen**: multi-agent dialogue collaboration + human-in-the-loop.

---

## LangGraph Architecture

---

## LangGraph workflow

---

## LangGraph Implementation
- blueprint.md
- claims_registry.yaml
- LangGraph_agent.py

### blueprint.md file
<font color=red>tell AI the work tasks</font>, which contains 6 necessary modules:
1. project: title + work_mission + one-sentence summary.
2. problem frameing: what problem are we going to solve? why existing methods fail?
3. method identity (most important).
4. core concepts (must locked): forbid AI to modify core words.
5. hard constraints: what must keep, add, and avoid.
6. experiment requirements: prevent AI to write useless experiments.

### claims_registry.yaml
tell AI the work constraints, which structure is follows:
- project: work_title
- global constraints
- claims:
  - id + label
  - claim description
  - type: empirical/method/framing/comparison, tell agent how to validate.
  - support_required (key): what experiments must do.
  - evidence_plan (design experiments): how to do experiments detailly.
  - risk (review view): reviwer thinking.
- reviewer_checklist: simulate a reviewer attack.
- consistency pass: check whether the logic is consistent before and after, preventing over-claim, claiming without validation, method drifting. <font color=red>It can be splited and merged to claims_registry file</font>.

### LangGraph_agent_v1.py
LangGraph code is like StateGraph(AgentState): add_nodes first, and then add_edges to connect them to form a graph pipeline. For example, 6 key stages following:  
\- read current_paper, blueprint, and claims  
-> summary blueprint  
-> formalize method  
-> design experiments  
-> work_1  
-> reviewer simulation + consistency_check.

Explain key codes:
- StateGraph(AgentState): LangGraph shared state container. Meaning that produced content of former node will be directly used by later node, connecting each agent with shared state.
- Summarize_blueprint: compress blueprint to a more suitable and brief impelmentation. input: claims.yaml + blueprint. output: bullet summary, hard constraints, reviewer risks, one-sentence core idea. why? Because a whole bunch of things in blueprint directly input to later nodes would generate chaos, compressed first and then input would be more stable.
- formalize method: work formalization. input: paper + blueprint_summary + hard constraints. output: method identity, pipeline stages, claims.
- design experiments: input: blueprint_summary + method_spec + claims.yaml; output: experiments (datasets, baselines, metrics, ablations, diagnostics, tables).
- work_1: starter for test. input: paper + blueprint_summary + method_spec + experiment_plan + hard_constraints; output is work_1 output.
- reviewer_simulation. simulate reviewer to attack work_1 output. input: blueprint_summary + method_spec + experiment_plan + work_1 output; output: concern + severity + exact fix.
- consistency_check: control agent output quality. input: blueprint_summary + method_spec + experiment_plan + work_1 output + reviewer comments; output: pass/fail, blocking issues, recommended next node, revision checklist.
- llm().invoke(prompt).content: create a llm client to invoke a unified API to input prompt and get a message object return, and then message.content is model output plain text.

### Running LangGraph agents
1. install packages  
  > pip install langgraph  
    pip install langgraph-ollama

2. run LangGraph  
  > python LangGraph.py   

---

## Final_LangGraph_Agent_v2
\+ conditional rollback + subgraph formalizer  
=> shared_state + conditional rollback + LLM API + checkpoint + manually check + section-by-section upgrade + consistency-driven loop
- why do we need section-by-section work? Because each work object is different.  
-consistency_check(): final's one of most important abilities, output report: Pass, Fail, or Revise.  
    ↓
- parse_route(): extract decision from consistency_report, route = parse_route(report).  
    ↓
- route_after_consistency(): <font coloar=red>LangGraph contral center.</font>
    - if consistency_route_decision == "approve": -> final_bundle_output.
    - if revision >= Max_revision_loops: -> return human_review_node.
- human_review_node(): final's new important ability. when automatic revision/consistency_report fail, pause graph and wait for human input instructions.  
    ↓
- apply_human_feedback(): inject human opinions to method and experiment plan.  
    ↓
- revise_work(): when consistency_report output fail decision, not revise whole paper, but section modification.  
    ↓
- revision_loop(): put revise_work again to reviewer attack and consistency check.
