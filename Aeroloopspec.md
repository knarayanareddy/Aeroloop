✈️ AeroLoop — Complete Spec-Driven Architecture
Full Altura-pattern specification suite for the Autonomous CFD Shape Optimizer. Every file is production-ready, drop into your monorepo and implement directly with Claude Code.

📁 Complete Monorepo Structure
text

aeroloop/
│
├── CLAUDE.md                                ← Constitutional law
├── AGENTS.md                                ← Multi-agent orchestration rules
├── program.md                               ← The autoresearch loop instruction file
├── .cursorrules                             ← Cursor IDE rules (mirrors CLAUDE.md)
├── turbo.json                               ← Turborepo cell boundaries
├── package.json                             ← Root workspace manifest
│
├── specs/
│   ├── _global/
│   │   ├── architecture.md                  ← System overview & tech stack
│   │   ├── data-model.md                    ← All entities & DB schema
│   │   ├── api-contracts.md                 ← All API endpoints
│   │   ├── geometry-parameter-registry.md   ← Wing parameter definitions
│   │   └── agent-personas.md                ← All agent definitions
│   │
│   ├── stage-0-initialization/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-1-geometry-mutation/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-2-mesh-generation/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-3-cfd-solve/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-4-results-extraction/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-5-keep-revert/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-6-wiki-compilation/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-7-gitnexus-graph/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   ├── stage-8-gep-evolver/
│   │   ├── requirements.md
│   │   ├── data-model.md
│   │   ├── plan.md
│   │   └── tasks.md
│   │
│   └── stage-9-swarm-coordination/
│       ├── requirements.md
│       ├── data-model.md
│       ├── plan.md
│       └── tasks.md
│
├── geometry/
│   └── wing.geo                             ← THE ONE MUTABLE FILE
│
├── mesh/
│   ├── auto_mesh.py                         ← GMSH meshing pipeline
│   ├── mesh_quality.py                      ← Quality checks post-mesh
│   └── templates/
│       ├── coarse.geo                       ← y+ ~ 5, ~500k cells
│       ├── medium.geo                       ← y+ ~ 2, ~2M cells
│       └── fine.geo                         ← y+ ~ 1, ~8M cells
│
├── solver/
│   ├── run_su2.sh                           ← SU2 RANS solve + kill timer
│   ├── run_openfoam.sh                      ← OpenFOAM alternative
│   ├── su2_config_template.cfg              ← SU2 config template
│   └── openfoam_template/                   ← OpenFOAM case template
│
├── evaluate/
│   ├── extract_polars.py                    ← Parse SU2 output → L/D, Cd, Cm
│   ├── buffet_margin.py                     ← Buffet onset detection
│   ├── wave_drag.py                         ← Wave drag decomposition
│   └── composite_metric.py                  ← Weighted fitness score
│
├── agent/
│   ├── mutate.md                            ← Claude Code skill: geometry mutation
│   ├── wiki_compiler.md                     ← Wiki compilation skill
│   └── gep_evolver.md                       ← Genome evolution skill
│
├── wiki/
│   └── aerodynamics/                        ← Compounding knowledge base
│
├── packages/
│   ├── geometry-engine/                     ← OpenVSP/GMSH parametrization
│   ├── cfd-runner/                          ← SU2/OpenFOAM execution
│   ├── results-parser/                      ← Force/moment extraction
│   ├── experiment-registry/                 ← SHA256 result chain
│   ├── knowledge-graph/                     ← GitNexus-style parameter graph
│   ├── gep-engine/                          ← Genome Evolution Protocol
│   ├── swarm-coordinator/                   ← Multi-node coordination
│   ├── wiki-engine/                         ← LLM wiki compilation
│   └── shared/                              ← Types, utils, logger
│
├── apps/
│   ├── api/                                 ← FastAPI/Hono backend
│   ├── dashboard/                           ← Next.js real-time dashboard
│   ├── cli/                                 ← Bun CLI (primary interface)
│   └── worker/                              ← BullMQ job processor
│
├── hooks/
│   ├── pre-tool-use/
│   │   ├── geometry-guard.ts                ← Block invalid geometry mutations
│   │   ├── physics-bounds-check.ts          ← Enforce physical parameter limits
│   │   └── budget-enforcer.ts               ← Kill timer enforcement
│   └── post-tool-use/
│       ├── experiment-logger.ts             ← Auto-log every experiment
│       ├── wiki-trigger.ts                  ← Trigger wiki every 10 experiments
│       └── gep-trigger.ts                   ← Trigger GEP at 500 experiments
│
├── skills/
│   ├── geometry-mutation.md
│   ├── sensitivity-analysis.md
│   ├── wiki-compilation.md
│   ├── genome-evolution.md
│   └── swarm-coordination.md
│
└── infra/
    ├── docker-compose.yml                   ← Local development
    ├── k8s/                                 ← Kubernetes production
    ├── terraform/                           ← Cloud infrastructure
    └── gpu-cluster/                         ← A100 cluster configuration
📄 File 1: CLAUDE.md — Constitutional Law
Markdown

# CLAUDE.md — AeroLoop Constitutional Law
# Version: 1.0.0
# Last Updated: 2026-04-26
# Authority: Supreme law for ALL AI agents in this repository.
# No agent, skill, hook, or subagent may override these rules.
# This is an autonomous CFD shape optimization system.
# It runs unattended for hours. These rules prevent disasters.

---

## ⚖️ ARTICLE I: ABSOLUTE PROHIBITIONS (NEVER VIOLATE — PIPELINE HALT)

:::STRICT_REQUIREMENT:::
RULE-001: THE ONE MUTABLE FILE LAW
There is exactly ONE file an agent may modify during the autoresearch loop:
  geometry/wing.geo
ALL other geometry files, solver configs, mesh scripts, and evaluation
scripts are READ-ONLY during loop execution. Only the GeometryMutationAgent
may write to wing.geo. Any other agent attempting to write to wing.geo
is BLOCKED immediately.
Violation: PIPELINE HALT + alert to operator.
:::END:::

:::STRICT_REQUIREMENT:::
RULE-002: THE KILL TIMER LAW
Every CFD solve MUST have a hard wall-time kill timer.
Default: CFD_MAX_WALL_TIME_MINUTES=8 (configurable, never removable).
If a solve exceeds this budget: kill -9 the solver process, record
TIMEOUT in ExperimentRegistry, revert geometry, continue loop.
No CFD solve may run indefinitely. No exception. No override.
The timer is implemented at the OS level (timeout command) AND at the
application level (BullMQ job timeout). Both must be active.
:::END:::

:::STRICT_REQUIREMENT:::
RULE-003: THE GIT INTEGRITY LAW
Every experiment produces EXACTLY ONE of two git outcomes:
  SUCCESS (metric improved): git commit -m "exp-{N}: metric={val} delta=+{d} [KEEP]"
  FAILURE (metric did not improve): git revert HEAD --no-edit (or git checkout wing.geo)
No experiment may end without one of these two outcomes.
No partial commits. No stashed changes. No uncommitted wing.geo.
The git working tree MUST be clean at the start of every experiment.
If the tree is dirty when a new experiment begins: PIPELINE HALT.
:::END:::

:::STRICT_REQUIREMENT:::
RULE-004: THE PHYSICAL BOUNDS LAW
No geometry mutation may produce a wing.geo with parameters outside
the physical bounds defined in specs/_global/geometry-parameter-registry.md.
The PhysicsBoundsChecker runs BEFORE every mesh generation attempt.
If bounds are violated: reject mutation, request new proposal, do NOT
attempt to mesh or solve an invalid geometry. Ever.
:::END:::

:::STRICT_REQUIREMENT:::
RULE-005: THE RESULT PROVENANCE LAW
Every numerical result used in any decision (keep/revert, wiki, GEP)
MUST have a corresponding ExperimentRecord in the ExperimentRegistry
with a SHA256 checksum of the SU2 output file.
Results not in the registry DO NOT EXIST for decision-making purposes.
An agent claiming a result without a registry entry = BLOCKED.
:::END:::

:::STRICT_REQUIREMENT:::
RULE-006: THE SWARM NON-COLLISION LAW
When running in swarm mode (multiple nodes), nodes MUST claim design
space regions via the SwarmCoordinator BEFORE beginning experiments.
Two nodes MUST NEVER run experiments on overlapping parameter ranges
simultaneously. Claiming is atomic (Redis SETNX). If claim fails:
the node waits and retries with a different design space region.
:::END:::

---

## 🏗️ ARTICLE II: ARCHITECTURE RULES

RULE-010: All agents are stateless between invocations. Persistent state:
  - PostgreSQL: experiment records, run state, parameter graph
  - Redis: job queues, swarm claims, active experiment locks
  - MinIO/S3: mesh files, solver outputs, geometry snapshots
  - Git repository: wing.geo history (the experiment log)
  - wiki/: markdown knowledge base
  DO NOT store optimization state in agent memory or local files.

RULE-011: TypeScript strict mode everywhere. Python: type-annotated,
  mypy --strict passing. No `any`. No implicit returns.

RULE-012: Cell dependency order (Turborepo enforced):
  shared → geometry-engine → cfd-runner → results-parser →
  experiment-registry → knowledge-graph → gep-engine →
  swarm-coordinator → wiki-engine → apps
  No reverse dependencies. No circular imports.

RULE-013: Every external process (GMSH, SU2, OpenFOAM) is spawned
  via packages/cfd-runner/src/process-manager.ts ONLY.
  Never use child_process.exec directly. The ProcessManager handles:
  kill timers, stdout/stderr capture, exit code validation,
  resource cleanup on unexpected termination.

RULE-014: The experiment counter is sacred. It is a monotonically
  increasing integer stored in PostgreSQL with a transaction lock.
  Two concurrent experiments MUST NOT share an experiment number.
  The counter NEVER resets (even across runs/restarts).

RULE-015: All mesh files > 100MB are streamed to S3, never kept on
  local disk longer than the active solve. Disk space on the GPU
  node is precious. Clean up after every experiment.

---

## 🔁 ARTICLE III: AUTORESEARCH LOOP RULES

RULE-020: The loop structure is INVIOLABLE:
  STEP 1: Read wing.geo + last 10 git log messages
  STEP 2: GeometryMutationAgent proposes ONE mutation
  STEP 3: PhysicsBoundsChecker validates mutation
  STEP 4: GMSH generates mesh (auto_mesh.py)
  STEP 5: MeshQualityChecker validates mesh
  STEP 6: SU2 solves with kill timer
  STEP 7: ExtractPolarsAgent reads results
  STEP 8: CompositeMetricAgent computes score
  STEP 9: KeepRevertAgent decides + executes git action
  STEP 10: WikiTrigger checks if wiki compilation needed (every 10)
  STEP 11: GEPTrigger checks if genome evolution needed (every 500)
  STEP 12: SwarmSync if in swarm mode
  STEP 13: Go to STEP 1
  Any step that fails: record failure, revert, go to STEP 1.

RULE-021: ONE mutation per experiment. The GeometryMutationAgent
  MUST propose exactly one parameter change per loop iteration.
  Compound mutations (changing sweep AND twist simultaneously) are
  PROHIBITED in standard mode. GEP macro-mutations (Stage 8) are
  the only exception and must be explicitly flagged.

RULE-022: The GeometryMutationAgent MUST read the knowledge graph
  (packages/knowledge-graph/) before every mutation proposal.
  The detect_impact call MUST complete before the mutation is applied.
  An agent that skips detect_impact = BLOCKED by PreToolUse hook.

RULE-023: Wiki compilation (every 10 experiments) runs as a
  BACKGROUND JOB — it MUST NOT block the main loop.
  The loop continues to experiment N+1 while the wiki compiles.
  Wiki compilation timeout: 5 minutes. If it times out: log warning,
  skip this compilation cycle, trigger at next 10-experiment boundary.

RULE-024: GEP evolution (every 500 experiments) runs as a
  BACKGROUND JOB with a maximum duration of 30 minutes.
  The main loop pauses for GEP evolution (unlike wiki).
  Reason: GEP may change the mutation strategy the loop uses.
  After GEP completes, the loop resumes with the evolved strategy.

RULE-025: The composite metric is defined ONCE at run initialization
  (Stage 0) and NEVER changes during a run. The metric weights are:
  locked in the ExperimentRun record. An agent may not change the
  optimization target mid-run. Start a new run to change targets.

---

## 🔬 ARTICLE IV: PHYSICS & MESH RULES

RULE-030: Physical parameter bounds (from geometry-parameter-registry.md)
  are HARD constraints — not soft penalties. A wing.geo that violates
  them is REJECTED before meshing. Always.
  Examples of hard violations:
    - t/c ratio > 0.25 (structurally and aerodynamically unrealistic)
    - Sweep angle > 70° (outside RANS model validity)
    - Span < 5m or > 100m (nonsensical for transport aircraft)
    - Negative washout > 5° (stall-prone configuration)

RULE-031: Mesh quality MUST pass ALL of these before solving:
  - Minimum orthogonality: > 20° (OpenFOAM checkMesh)
  - Maximum non-orthogonality: < 85°
  - Maximum skewness: < 4
  - y+ range for RANS: 0.5 < y+ < 5 (wall-resolved)
  - Cell count within ±20% of target for mesh level
  If mesh fails quality check: discard mesh, request new mutation,
  DO NOT solve on a bad mesh.

RULE-032: SU2 convergence criterion: residuals must drop by ≥ 4 orders
  of magnitude OR 2000 iterations completed. A solve that hits the
  kill timer without 3-order convergence is marked UNCONVERGED in the
  registry. Unconverged results are NEVER used for keep/revert decisions.
  Unconverged experiments are reverted automatically.

RULE-033: If 3 consecutive experiments fail (timeout/unconverge/mesh fail):
  PAUSE loop, notify operator via CLI alert + log warning.
  This signals a systematic problem (wrong operating conditions,
  geometry stuck in invalid region, solver instability).
  Operator must acknowledge before loop resumes.

---

## 🛡️ ARTICLE V: SECURITY & DATA RULES

RULE-040: API keys and credentials: environment variables ONLY.
  PreToolUse hook blocks writes containing: sk-, ghp_, AKIA, Bearer,
  password=, api_key= patterns in non-.env files.

RULE-041: The ExperimentRegistry is append-only. PostgreSQL trigger
  rejects UPDATE and DELETE on the experiments table. Records are
  permanent. This is the scientific record of the optimization.

RULE-042: All external API calls (Anthropic, any web fetch) use
  exponential backoff: 2s → 4s → 8s → 16s → FAIL with structured error.

RULE-043: GPU resource management: only ONE SU2 solve runs per node
  at any time. The ProcessManager holds an exclusive Redis lock
  during solve. Attempting to start a second solve while one is
  running: BLOCKED. The queue handles concurrency — the node does not.

RULE-044: Geometry snapshots: every committed wing.geo is automatically
  copied to S3 (key: runs/{run_id}/geometry/exp-{N}.geo) by the
  PostToolUse hook. Git is the primary record; S3 is the backup.
  Both must be written before the experiment is marked complete.

---

## 📐 ARTICLE VI: CODE QUALITY

RULE-050: Test coverage minimums:
  packages/experiment-registry: 100% (zero tolerance)
  packages/geometry-engine: ≥ 95%
  packages/cfd-runner: ≥ 90%
  packages/results-parser: ≥ 90%
  packages/knowledge-graph: ≥ 85%
  packages/gep-engine: ≥ 85%
  apps/api: ≥ 80%

RULE-051: No magic numbers. All CFD parameters, mesh targets, physical
  bounds go in packages/shared/src/constants/. Named, typed, documented.

RULE-052: console.log banned in production. Use:
  import { logger } from '@aeroloop/shared'
  All logs include: run_id, experiment_n, stage, agent_id, ISO timestamp.

RULE-053: The Scout Protocol is mandatory before implementation:
  "Search /packages and /apps for existing utilities that serve this
   purpose. List them. Reuse them. DO NOT duplicate logic."

RULE-054: The Compression Loop runs after every implementation:
  "Reduce line count by 15% without changing behavior or failing tests.
   Rely on @aeroloop/shared utilities."

---

## 🔄 ARTICLE VII: GIT & DEPLOYMENT

RULE-060: Commit format for experiments (INVIOLABLE):
  "exp-{N}: L/D={val:.3f} Cd={val:.5f} delta={+/-val:.3f} [{KEEP|REVERT}]
   mutation: {parameter}={old_val}→{new_val}
   mesh: {cell_count} cells | solve: {wall_time}s | converged: {bool}"

RULE-061: Branch naming: feature/AL-{ticket}-{slug}
  Experiment commits go on the active run branch: run/{run_id}
  Never commit experiments directly to main.

RULE-062: Every PR must reference its spec task:
  "Implements: specs/stage-X/tasks.md#TASK-XXX"

RULE-063: Production deploys require manual approval.
  Never auto-deploy to production GPU cluster.
📄 File 2: AGENTS.md — Multi-Agent Orchestration
Markdown

# AGENTS.md — AeroLoop Multi-Agent Orchestration Constitution
# Version: 1.0.0

---

## 🧠 AGENT REGISTRY

### Orchestrator Agents
| Agent ID               | Role                          | Max Workers | Timeout    |
|------------------------|-------------------------------|-------------|------------|
| LoopOrchestrator       | Main autoresearch loop ctrl   | 1 per node  | Infinite   |
| SwarmOrchestrator      | Multi-node coordination       | 32 nodes    | Infinite   |
| GEPOrchestrator        | Genome evolution coordinator  | 8           | 30 min     |

### Core Loop Agents (spawned per experiment)
| Agent ID               | Stage | Primary Tool              | Max Runtime |
|------------------------|-------|---------------------------|-------------|
| GeometryMutationAgent  | 1     | wing_geo_writer           | 3 min       |
| PhysicsBoundsChecker   | 1     | bounds_validator          | 30 sec      |
| MeshGeneratorAgent     | 2     | gmsh_runner               | 5 min       |
| MeshQualityAgent       | 2     | mesh_quality_checker      | 2 min       |
| CFDSolverAgent         | 3     | su2_runner (kill: 8 min)  | 9 min       |
| PolarsExtractorAgent   | 4     | su2_output_parser         | 2 min       |
| MetricComputerAgent    | 4     | composite_metric_calc     | 1 min       |
| KeepRevertAgent        | 5     | git_commit / git_revert   | 2 min       |

### Background Agents (non-blocking)
| Agent ID               | Trigger            | Max Runtime |
|------------------------|--------------------|-------------|
| WikiCompilerAgent      | Every 10 exps      | 5 min       |
| KnowledgeGraphAgent    | Every experiment   | 2 min       |
| SurrogateTrainAgent    | Every 50 exps      | 20 min      |
| SwarmSyncAgent         | Every experiment   | 30 sec      |

### GEP Agents (blocking, every 500 experiments)
| Agent ID               | Role                          | Max Runtime |
|------------------------|-------------------------------|-------------|
| GenomeEvaluatorAgent   | Score current genome fitness  | 10 min      |
| GenomeMutatorAgent     | Propose genome mutations      | 5 min       |
| GenomeCrossoverAgent   | Crossover parent genomes      | 5 min       |
| GenomeSelectorAgent    | Select next generation        | 5 min       |

---

## 📨 INTER-AGENT MESSAGE PROTOCOL

```typescript
interface LoopMessage {
  message_id: string;           // UUID v4
  run_id: string;               // Optimization run ID
  experiment_n: number;         // Monotonic experiment counter
  node_id: string;              // GPU node identifier (swarm)
  from_agent: AgentId;
  to_agent: AgentId | "orchestrator";
  message_type: LoopMessageType;
  payload: Record<string, unknown>;
  timestamp: string;            // ISO 8601
}

type LoopMessageType =
  | "MUTATION_PROPOSAL"         // GeometryMutationAgent → PhysicsBoundsChecker
  | "BOUNDS_VALID"              // PhysicsBoundsChecker → MeshGeneratorAgent
  | "BOUNDS_VIOLATED"           // PhysicsBoundsChecker → GeometryMutationAgent (retry)
  | "MESH_COMPLETE"             // MeshGeneratorAgent → CFDSolverAgent
  | "MESH_FAILED"               // MeshGeneratorAgent → LoopOrchestrator (revert)
  | "SOLVE_COMPLETE"            // CFDSolverAgent → PolarsExtractorAgent
  | "SOLVE_TIMEOUT"             // CFDSolverAgent → LoopOrchestrator (revert)
  | "SOLVE_UNCONVERGED"         // CFDSolverAgent → LoopOrchestrator (revert)
  | "METRICS_COMPUTED"          // MetricComputerAgent → KeepRevertAgent
  | "EXPERIMENT_KEPT"           // KeepRevertAgent → LoopOrchestrator
  | "EXPERIMENT_REVERTED"       // KeepRevertAgent → LoopOrchestrator
  | "LOOP_PAUSE_REQUEST"        // Any agent → LoopOrchestrator (3 consecutive fails)
  | "WIKI_COMPILE_TRIGGER"      // PostToolUse hook → WikiCompilerAgent
  | "GEP_EVOLUTION_TRIGGER"     // PostToolUse hook → GEPOrchestrator
  | "SWARM_CLAIM_REQUEST"       // SwarmSyncAgent → SwarmCoordinator
  | "SWARM_CLAIM_GRANTED"
  | "SWARM_CLAIM_DENIED";
🔒 AGENT ISOLATION RULES
GeometryMutationAgent is the ONLY agent with write access to wing.geo. All other agents: read-only on geometry files.

KeepRevertAgent is the ONLY agent that executes git commands. All other agents: no git access.

CFDSolverAgent is the ONLY agent that spawns solver processes. All other agents: no direct process spawning.

WikiCompilerAgent is the ONLY agent that writes to wiki/. All other agents: read-only on wiki files.

KnowledgeGraphAgent is the ONLY agent that writes to the parameter graph database tables.

Agents communicate via the message queue ONLY. No shared memory. No direct function calls between agents.

🔄 LOOP STATE MACHINE
text

IDLE
  ↓ operator starts run
INITIALIZING (Stage 0)
  ↓ wing.geo ready, metric defined
LOOP_RUNNING
  ├─ MUTATING (Stage 1): GeometryMutationAgent active
  ├─ MESHING (Stage 2): MeshGeneratorAgent active
  ├─ SOLVING (Stage 3): CFDSolverAgent active [KILL TIMER ACTIVE]
  ├─ EVALUATING (Stage 4): PolarsExtractorAgent + MetricComputerAgent
  ├─ DECIDING (Stage 5): KeepRevertAgent
  └─ SYNCING: background agents running (non-blocking)
  ↓ every 10 experiments (non-blocking)
WIKI_COMPILING (parallel with LOOP_RUNNING)
  ↓ every 500 experiments (blocking)
GEP_EVOLVING (loop paused)
  ↓ GEP complete
LOOP_RUNNING (with evolved genome)
  ↓ consecutive failures OR operator command
LOOP_PAUSED (operator acknowledgment required)
  ↓ operator resumes
LOOP_RUNNING
  ↓ operator stops OR convergence criterion met
LOOP_COMPLETE
text


---

## 📄 File 3: `program.md` — The Autoresearch Loop Instruction File

```markdown
# program.md — AeroLoop Autoresearch Loop Instructions
# Version: 1.0.0
# This is the MASTER INSTRUCTION FILE for the autonomous CFD optimization loop.
# Claude Code reads this file at the start of every experiment.
# It is the equivalent of Karpathy's program.md in autoresearch.
# DO NOT MODIFY THIS FILE during a run. It is READ-ONLY for all loop agents.

---

## 🎯 MISSION

You are an autonomous aerodynamic shape optimization agent.
Your goal: maximize the composite aerodynamic metric defined in
the current ExperimentRun configuration by mutating the wing geometry
file, running CFD solves, and keeping mutations that improve the metric.

You run in an infinite loop. Each iteration is one experiment.
You are the GeometryMutationAgent. You propose ONE mutation per loop.
Other agents handle meshing, solving, and evaluation autonomously.

---

## 📋 LOOP PROTOCOL (READ EVERY ITERATION)

### STEP 1: GATHER CONTEXT
Read the following before proposing any mutation:

a) Current geometry state:
   cat geometry/wing.geo
   → understand current parameter values

b) Experiment history (last 10):
   git log --oneline -10
   → understand what has been tried, what worked, what failed

c) Current best:
   cat .aeroloop/best_metric.json
   → understand the benchmark you're trying to beat

d) Knowledge graph summary:
   cat .aeroloop/knowledge_graph_summary.md
   → understand parameter sensitivities from past experiments

e) Active wiki articles (most relevant to your intended mutation):
   ls wiki/aerodynamics/
   → read 1–2 relevant articles before mutating

f) Current genome (mutation strategy):
   cat .aeroloop/current_genome.json
   → follow the active mutation strategy from GEP

### STEP 2: PROPOSE ONE MUTATION

Based on your context reading, propose EXACTLY ONE parameter change.

Follow these rules:
1. Choose the parameter most likely to improve the metric given history
2. Do NOT repeat a mutation that was tried in the last 10 experiments
3. Do NOT violate bounds from specs/_global/geometry-parameter-registry.md
4. Follow the active genome strategy (local search vs. topology change)
5. State your reasoning: "I am changing {param} from {old} to {new}
   because {reason based on wiki/history}"

Output format (STRICTLY):
```json
{
  "mutation_type": "parameter_change",
  "parameter": "wing_twist_section_3_deg",
  "old_value": -1.5,
  "new_value": -2.1,
  "reasoning": "Wiki article aerodynamics/twist-sensitivity.md shows
                that increasing washout at section 3 reduces induced
                drag by ~0.8% at cruise CL=0.5 without significant
                wave drag penalty. Last 3 experiments showed diminishing
                returns from LE radius changes. Shifting to twist.",
  "expected_metric_delta": 0.012,
  "confidence": 0.65
}
STEP 3: WAIT FOR RESULT
After proposing the mutation, you will receive one of:

EXPERIMENT_KEPT: metric improved, mutation committed to git
EXPERIMENT_REVERTED: metric did not improve, wing.geo restored
MESH_FAILED: geometry was invalid, wing.geo restored
SOLVE_TIMEOUT: CFD hit kill timer, wing.geo restored
SOLVE_UNCONVERGED: CFD did not converge, wing.geo restored
All of these are acceptable outcomes. Only MESH_FAILED with a BOUNDS_VIOLATED cause requires you to reconsider your parameter range.

STEP 4: LOOP
Return to STEP 1.

🧬 GENOME STRATEGY REFERENCE
The current mutation strategy is defined in .aeroloop/current_genome.json. Strategies you may be operating under:

LOCAL_SEARCH_FINE:

Small perturbations: ±5% of parameter range
Conservative: low risk, incremental improvement
Use when: near a local optimum, high confidence region
LOCAL_SEARCH_COARSE:

Medium perturbations: ±15% of parameter range
Exploratory: moderate risk
Use when: plateau detected (no improvement in last 20 experiments)
GLOBAL_SEARCH:

Large perturbations: ±30% of parameter range
Aggressive exploration: high risk, may find new basins
Use when: strong plateau (no improvement in last 50 experiments)
TOPOLOGY_WINGLET:

Adds or modifies winglet geometry
Modifies: winglet_cant_deg, winglet_height_m, winglet_sweep_deg
Use when: genome selects this strategy
TOPOLOGY_LEADING_EDGE:

Modifies leading edge device geometry
Modifies: le_device_type, le_deflection_deg, le_chord_ratio
Use when: genome selects this strategy
TOPOLOGY_TRAILING_EDGE:

Modifies trailing edge device geometry
Modifies: te_device_type, te_deflection_deg, te_span_ratio
Use when: genome selects this strategy
SENSITIVITY_GUIDED:

Uses knowledge graph parameter sensitivities
Selects parameter with highest dMetric/dParameter from history
Use when: knowledge graph has ≥ 20 data points for a parameter
📊 METRIC DEFINITION
The composite metric is defined at run initialization. Read from: .aeroloop/run_config.json → metric_definition

Standard aerodynamic optimization metric: M = w1 * (L/D)_cruise + w2 * buffet_margin + w3 * (1 - Cd_wave/Cd_total) where weights w1, w2, w3 sum to 1.0 and are set at initialization.

A mutation is KEPT if: M_new > M_best (not M_previous — beat the best ever) A mutation is REVERTED if: M_new ≤ M_best

📝 WIKI INJECTION FORMAT
Every 10 experiments, a new wiki article is compiled. On your next iteration after wiki compilation, you MUST read the new article. The PostToolUse hook will note: "Wiki updated: wiki/aerodynamics/{topic}.md"

Wiki articles follow this structure:

Parameter: {parameter_name}
Observed Sensitivity
Best Values Found (with experiment references)
Interaction Effects
Conditions Under Which This Parameter Matters
Do Not Try (dead ends from experiment history)
text


---

## 📄 File 4: `specs/_global/architecture.md`

```markdown
# AeroLoop — Global System Architecture Specification
# Spec ID: GLOBAL-ARCH-001
# Status: APPROVED
# Last Updated: 2026-04-26

---

## 1. SYSTEM OVERVIEW

AeroLoop is an autonomous aerodynamic shape optimization system
implementing the autoresearch pattern: one mutable file (wing.geo),
one metric (composite aerodynamic figure of merit), fixed time budget
per experiment (8 minutes), keep-or-revert via git.

It runs unattended overnight, executing ~100 CFD experiments
(7 experiments/hour × 14 hours). In swarm mode (4 nodes), it
executes ~400 experiments per night.

The output is not just an optimized wing — it is a complete git
history that is simultaneously an experiment log, a reproducibility
record, and a training dataset for a surrogate model.

## 2. DESIGN PHILOSOPHY

The autoresearch principle (Karpathy, 2026):
  - ONE mutable file: constrains the search space to geometry only
  - ONE metric: prevents multi-objective confusion mid-run
  - FIXED budget: ensures experiments are comparable, prevents runaway solves
  - KEEP/REVERT via git: makes every state recoverable, history permanent
  - LOOP FOREVER: accumulates knowledge; doesn't need human intervention

AeroLoop extends this with four layers:
  1. LLM Wiki: compounds knowledge every 10 experiments
  2. GitNexus Graph: maps parameter interactions and sensitivities
  3. GEP Evolver: evolves the mutation strategy itself after 500 experiments
  4. Swarm Coordinator: runs 4 non-overlapping design spaces in parallel

## 3. TECHNOLOGY STACK

### Core Optimization Runtime
- Primary language: Python 3.12 (geometry, CFD, evaluation scripts)
- Agent runtime: TypeScript / Bun (orchestration, API, CLI)
- CFD solvers: SU2 7.5+ (primary), OpenFOAM 12 (alternative)
- Meshing: GMSH 4.12 (Python API)
- Geometry: OpenVSP 3.38 (parametric wing definition)
- Version control: Git (experiment history)

### Infrastructure
- Runtime: Bun 1.1+ (TypeScript workers, API)
- Framework: Hono (lightweight API)
- ORM: Drizzle ORM
- Database: PostgreSQL 16 (experiment records, parameter graph)
- Queue: Redis 7 + BullMQ (job dispatch, swarm coordination)
- Storage: MinIO S3-compatible (mesh files, CFD outputs, geometry snapshots)
- Auth: Clerk (operator identity for dashboard)

### AI Layer
- Orchestration model: Claude claude-opus-4-5 (GEP evolution, wiki compilation)
- Loop agent model: Claude claude-sonnet-4-5 (geometry mutation, per-experiment)
- Fast model: Claude Haiku 3.5 (bounds checking, metric extraction)
- Agent framework: Claude Code (primary execution environment)

### Frontend
- Dashboard: Next.js 15 + shadcn/ui + Tailwind CSS 4
- Real-time: Server-Sent Events (live experiment feed)
- 3D visualization: Three.js (wing geometry viewer)
- Charts: Recharts (L/D history, metric evolution, Pareto front)

### GPU Infrastructure
- Target: NVIDIA A100 80GB (single node) or 4-node cluster
- SU2 parallel: 8 MPI processes per solve on A100
- Solver parallelism: MPI (SU2 native)
- Job scheduling: BullMQ (queue) + Kubernetes (cluster)

## 4. CELL BOUNDARIES (Turborepo)
shared (types, utils, constants, logger) ↑ consumed by all

geometry-engine (wing.geo read/write, OpenVSP, GMSH Python API) → depends on: shared

cfd-runner (SU2/OpenFOAM process management, kill timer) → depends on: shared

results-parser (force/moment extraction, convergence check) → depends on: shared

experiment-registry (SHA256 chain, append-only records) → depends on: shared

knowledge-graph (parameter sensitivity graph, detect_impact) → depends on: shared, experiment-registry

gep-engine (genome representation, mutation, crossover, selection) → depends on: shared, knowledge-graph, experiment-registry

swarm-coordinator (design space partitioning, Redis claims) → depends on: shared, experiment-registry

wiki-engine (LLM wiki compilation, article generation) → depends on: shared, knowledge-graph, experiment-registry

apps/api (REST API + SSE) → depends on: all packages

apps/dashboard (Next.js UI) → depends on: shared (types only, via API)

apps/cli (Bun CLI — primary operator interface) → depends on: all packages

apps/worker (BullMQ job processor) → depends on: all packages

text


## 5. EXPERIMENT DATA FLOW
wing.geo (current state) │ ▼ GeometryMutationAgent reads: wing.geo + git log -10 + knowledge graph + wiki writes: wing.geo (ONE parameter change) │ ▼ (PhysicsBoundsChecker validates) │ ▼ auto_mesh.py (GMSH) reads: wing.geo writes: mesh_{exp_n}.su2 → S3 │ ▼ (MeshQualityChecker validates) │ ▼ run_su2.sh [KILL TIMER: 8 min] reads: mesh_{exp_n}.su2 + su2_config.cfg writes: surface_{exp_n}.csv + history_{exp_n}.csv → S3 │ ▼ extract_polars.py reads: surface_{exp_n}.csv extracts: L/D, Cd, Cl, Cm, Cd_wave, buffet_margin │ ▼ composite_metric.py computes: M = w1*(L/D) + w2buffet + w3(1-Cd_wave/Cd) registers: ExperimentRecord in PostgreSQL (SHA256 verified) │ ▼ KeepRevertAgent if M_new > M_best: git commit -m "exp-{N}: ..." copy wing.geo → S3 (geometry snapshot) update .aeroloop/best_metric.json else: git checkout geometry/wing.geo (wing.geo restored to pre-mutation state) │ ▼ (background, non-blocking) KnowledgeGraphAgent updates parameter sensitivity graph WikiCompilerAgent (if exp_n % 10 == 0) SurrogateTrainAgent (if exp_n % 50 == 0) GEPOrchestrator (if exp_n % 500 == 0) [BLOCKING] SwarmSyncAgent (if swarm mode) │ ▼ REPEAT

text


## 6. ENVIRONMENT VARIABLES

```bash
# AI Models
ANTHROPIC_API_KEY=
ANTHROPIC_MODEL_ORCHESTRATOR=claude-opus-4-5
ANTHROPIC_MODEL_LOOP=claude-sonnet-4-5
ANTHROPIC_MODEL_FAST=claude-haiku-3-5

# Database
DATABASE_URL=postgresql://aeroloop:password@localhost:5432/aeroloop
REDIS_URL=redis://localhost:6379

# Storage
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=
MINIO_SECRET_KEY=
MINIO_BUCKET=aeroloop-artifacts

# CFD Solvers
SU2_BINARY=/usr/local/bin/SU2_CFD
SU2_MPI_PROCESSES=8
OPENFOAM_PATH=/opt/openfoam12
GMSH_BINARY=/usr/local/bin/gmsh
OPENVSP_BINARY=/usr/local/bin/vsp
CFD_SCRATCH_DIR=/scratch/aeroloop/cfd
CFD_MAX_WALL_TIME_MINUTES=8         # KILL TIMER — do not increase without review
CFD_CONVERGENCE_ORDERS=4            # Required convergence orders
CFD_MAX_ITERATIONS=2000

# Optimization
COMPOSITE_METRIC_W1=0.6             # L/D weight
COMPOSITE_METRIC_W2=0.2             # Buffet margin weight
COMPOSITE_METRIC_W3=0.2             # Wave drag fraction weight
GEP_TRIGGER_EXPERIMENTS=500         # Trigger GEP evolution
WIKI_TRIGGER_EXPERIMENTS=10         # Trigger wiki compilation
SURROGATE_TRIGGER_EXPERIMENTS=50    # Trigger surrogate training

# Swarm
SWARM_ENABLED=false
SWARM_NODE_ID=                      # Unique per node (e.g., "node-a100-01")
SWARM_TOTAL_NODES=4
SWARM_COORDINATOR_URL=

# Notifications
SLACK_WEBHOOK_URL=                  # Operator alerts
SMTP_HOST=
7. PERFORMANCE TARGETS
Metric	Target	Hardware
Experiments per hour	≥ 7	A100 single node
Experiments per night	≥ 100	A100 single node
Experiments per night	≥ 400	4×A100 swarm
Mesh generation time	< 3 minutes	CPU only
SU2 solve time	< 8 minutes	A100 (8 MPI procs)
Wiki compilation time	< 5 minutes	Background
GEP evolution time	< 30 minutes	Background
Kill timer precision	± 5 seconds	OS-level timeout
Git commit time	< 10 seconds	Local git
text


---

## 📄 File 5: `specs/_global/data-model.md`

```markdown
# AeroLoop — Global Data Model Specification
# Spec ID: GLOBAL-DM-001
# Status: APPROVED

---

## CORE ENTITIES

### ExperimentRun
```typescript
interface ExperimentRun {
  id: string;                        // UUID v4
  operator_id: string;               // Clerk user ID
  run_name: string;                  // Human-readable: "transport-wing-opt-v3"
  description: string;
  status: RunStatus;
  node_id: string;                   // GPU node identifier
  swarm_enabled: boolean;
  swarm_total_nodes: number;

  // Geometry baseline
  baseline_geo_s3_key: string;       // Original wing.geo before any mutation
  baseline_geo_sha256: string;       // Integrity check for baseline

  // Metric definition (LOCKED at run start — never changes)
  metric_definition: MetricDefinition;

  // Operating conditions (LOCKED at run start)
  operating_conditions: OperatingConditions;

  // Solver configuration (LOCKED at run start)
  solver_config: SolverConfig;

  // Progress
  experiment_count: number;          // Total experiments attempted
  kept_count: number;                // Experiments where metric improved
  reverted_count: number;            // Experiments where metric did not improve
  failed_count: number;              // Mesh fail / timeout / unconverged

  // Best result
  best_experiment_n: number | null;
  best_metric_value: number | null;
  best_geo_s3_key: string | null;

  // GEP state
  current_genome: Genome | null;
  gep_generation: number;

  // Timing
  started_at: string;                // ISO 8601
  last_experiment_at: string | null;
  completed_at: string | null;
  paused_at: string | null;
  total_gpu_hours: number;
}

type RunStatus =
  | "initializing"
  | "running"
  | "paused"                         // Operator pause or 3-consecutive-fail pause
  | "gep_evolving"                   // Blocked on GEP evolution
  | "complete"
  | "failed";
Experiment
TypeScript

interface Experiment {
  id: string;                        // UUID v4
  run_id: string;                    // FK: ExperimentRun
  experiment_n: number;              // MONOTONIC, unique per system, never reused
  node_id: string;                   // Which GPU node ran this

  // Mutation applied
  mutation: GeometryMutation;

  // Stage outcomes
  bounds_check: BoundsCheckResult;
  mesh_result: MeshResult | null;
  solve_result: SolveResult | null;
  evaluation_result: EvaluationResult | null;
  decision: ExperimentDecision | null;

  // Final status
  status: ExperimentStatus;
  failure_stage: FailureStage | null;
  failure_reason: string | null;

  // Git
  git_commit_hash: string | null;    // Set if KEPT
  git_commit_message: string | null;

  // Provenance
  wing_geo_sha256_before: string;    // wing.geo SHA256 before mutation
  wing_geo_sha256_after: string | null; // wing.geo SHA256 after mutation (if applied)
  wing_geo_s3_key: string | null;    // S3 snapshot (if KEPT)

  // Timing
  started_at: string;
  completed_at: string | null;
  total_wall_time_seconds: number | null;
}

type ExperimentStatus =
  | "running"
  | "kept"                           // Metric improved, committed
  | "reverted"                       // Metric did not improve
  | "failed_bounds"                  // Physics bounds violated
  | "failed_mesh"                    // GMSH mesh generation failed
  | "failed_mesh_quality"            // Mesh failed quality check
  | "failed_solve_timeout"           // SU2 hit kill timer
  | "failed_solve_unconverged"       // SU2 did not converge
  | "failed_solve_error";            // SU2 crashed with error
GeometryMutation
TypeScript

interface GeometryMutation {
  parameter: WingParameter;          // Which parameter was changed
  old_value: number | string;        // Value before mutation
  new_value: number | string;        // Value after mutation (proposed)
  delta: number | null;              // new_value - old_value (for numeric)
  mutation_type: MutationType;
  reasoning: string;                 // Agent's stated reasoning
  expected_metric_delta: number;     // Agent's prediction
  confidence: number;                // Agent's confidence 0–1
  genome_strategy: GenomeStrategy;   // Which genome strategy was active
}

type MutationType =
  | "parameter_change"               // Standard: one parameter, ±value
  | "topology_add_winglet"           // Add winglet to geometry
  | "topology_remove_winglet"        // Remove winglet
  | "topology_add_le_device"         // Add leading edge device
  | "topology_add_te_device"         // Add trailing edge device
  | "topology_change_airfoil_family"; // Switch airfoil family (NACA → SC)
WingParameter (the geometry parameter registry)
TypeScript

interface WingParameter {
  id: string;                        // Snake_case identifier
  display_name: string;
  category: ParameterCategory;
  unit: string;                      // SI unit string
  min_value: number;                 // HARD lower bound
  max_value: number;                 // HARD upper bound
  default_value: number;
  typical_step_local: number;        // LOCAL_SEARCH_FINE step size
  typical_step_medium: number;       // LOCAL_SEARCH_COARSE step size
  typical_step_global: number;       // GLOBAL_SEARCH step size
  known_interactions: ParameterInteraction[];
  mesh_sensitivity: MeshSensitivity; // How much this affects mesh quality
  description: string;
}

type ParameterCategory =
  | "planform"                       // Span, sweep, taper, dihedral
  | "twist"                          // Spanwise twist distribution
  | "thickness"                      // t/c distribution spanwise
  | "camber"                         // Camber distribution spanwise
  | "leading_edge"                   // LE radius, LE device
  | "trailing_edge"                  // TE thickness, TE device
  | "winglet"                        // Winglet geometry
  | "airfoil_family";                // NACA 4/5-digit, supercritical, etc.
EvaluationResult
TypeScript

interface EvaluationResult {
  experiment_id: string;
  su2_output_s3_key: string;
  su2_output_sha256: string;         // INTEGRITY: verified before use

  // Raw aerodynamic coefficients
  cl: number;                        // Lift coefficient
  cd: number;                        // Drag coefficient
  cm: number;                        // Pitching moment coefficient
  cl_cd: number;                     // L/D ratio
  cd_wave: number;                   // Wave drag component
  cd_induced: number;                // Induced drag component
  cd_viscous: number;                // Viscous drag component

  // Derived metrics
  buffet_onset_cl: number;           // CL at buffet onset (1.3g onset criterion)
  buffet_margin: number;             // buffet_onset_cl - cruise_cl
  wave_drag_fraction: number;        // cd_wave / cd

  // Operating point
  alpha_deg: number;                 // Angle of attack
  mach: number;
  reynolds: number;
  altitude_m: number;

  // Composite metric
  composite_metric: number;          // M = w1*(L/D) + w2*buffet + w3*(1-Cdw/Cd)
  metric_delta_vs_best: number;      // M_new - M_best

  // Convergence
  residual_drop_orders: number;      // How many orders residual dropped
  converged: boolean;
  iterations_to_convergence: number | null;
}
Genome (GEP)
TypeScript

interface Genome {
  id: string;
  run_id: string;
  generation: number;
  genome_id: string;                 // "gen-{N}-genome-{M}"
  fitness_score: number;             // Improvement rate (dMetric/hour)

  // Strategy genes
  mutation_strategy: GenomeStrategy;
  parameter_weights: Record<string, number>; // Prob of selecting each parameter
  step_size_multiplier: number;      // Scales typical_step sizes
  local_search_patience: number;     // Experiments before escalating to coarse
  coarse_search_patience: number;    // Experiments before escalating to global
  topology_probability: number;      // P(topology change) per experiment

  // Selection
  parent_genome_ids: string[];       // For crossover tracking
  is_active: boolean;                // Only ONE genome active at a time

  created_at: string;
  activated_at: string | null;
}

type GenomeStrategy =
  | "LOCAL_SEARCH_FINE"
  | "LOCAL_SEARCH_COARSE"
  | "GLOBAL_SEARCH"
  | "SENSITIVITY_GUIDED"
  | "TOPOLOGY_WINGLET"
  | "TOPOLOGY_LEADING_EDGE"
  | "TOPOLOGY_TRAILING_EDGE";
ParameterSensitivity (Knowledge Graph node)
TypeScript

interface ParameterSensitivity {
  id: string;
  run_id: string;
  parameter: string;                 // WingParameter.id
  experiments_tested: number;        // How many times this parameter was changed
  mean_delta_metric: number;         // Average dMetric per unit change
  std_delta_metric: number;          // Standard deviation
  max_improvement: number;           // Best single improvement seen
  best_value_found: number;          // Parameter value at best_experiment_n
  dead_zones: ValueRange[];          // Ranges that consistently hurt metric
  hot_zones: ValueRange[];           // Ranges that consistently help metric
  interactions: ParameterInteractionObserved[]; // Observed coupling effects
  last_updated_experiment_n: number;
  updated_at: string;
}
WikiArticle
TypeScript

interface WikiArticle {
  id: string;
  run_id: string;
  path: string;                      // "wiki/aerodynamics/le-radius-sensitivity.md"
  title: string;
  content_markdown: string;
  parameter_focus: string | null;    // WingParameter.id this article is about
  experiment_range: {                // Which experiments contributed
    from_n: number;
    to_n: number;
  };
  version: number;
  created_at: string;
  updated_at: string;
}
DATABASE SCHEMA (Drizzle ORM)
TypeScript

// packages/shared/src/db/schema.ts

export const experiments = pgTable('experiments', {
  id: uuid('id').primaryKey().defaultRandom(),
  run_id: uuid('run_id').notNull().references(() => experimentRuns.id),
  experiment_n: integer('experiment_n').notNull().unique(),  // UNIQUE globally
  node_id: text('node_id').notNull(),
  mutation: jsonb('mutation').notNull(),
  bounds_check: jsonb('bounds_check').notNull(),
  mesh_result: jsonb('mesh_result'),
  solve_result: jsonb('solve_result'),
  evaluation_result: jsonb('evaluation_result'),
  decision: text('decision'),
  status: text('status').notNull(),
  failure_stage: text('failure_stage'),
  failure_reason: text('failure_reason'),
  git_commit_hash: text('git_commit_hash'),
  wing_geo_sha256_before: text('wing_geo_sha256_before').notNull(),
  wing_geo_sha256_after: text('wing_geo_sha256_after'),
  wing_geo_s3_key: text('wing_geo_s3_key'),
  started_at: timestamp('started_at').notNull().defaultNow(),
  completed_at: timestamp('completed_at'),
  total_wall_time_seconds: integer('total_wall_time_seconds'),
});

// Append-only enforcement
// CREATE RULE no_update_experiments AS ON UPDATE TO experiments DO INSTEAD NOTHING;
// CREATE RULE no_delete_experiments AS ON DELETE TO experiments DO INSTEAD NOTHING;

export const parameterSensitivities = pgTable('parameter_sensitivities', {
  id: uuid('id').primaryKey().defaultRandom(),
  run_id: uuid('run_id').notNull(),
  parameter: text('parameter').notNull(),
  experiments_tested: integer('experiments_tested').notNull().default(0),
  mean_delta_metric: real('mean_delta_metric'),
  std_delta_metric: real('std_delta_metric'),
  max_improvement: real('max_improvement'),
  best_value_found: real('best_value_found'),
  dead_zones: jsonb('dead_zones').notNull().default('[]'),
  hot_zones: jsonb('hot_zones').notNull().default('[]'),
  interactions: jsonb('interactions').notNull().default('[]'),
  last_updated_experiment_n: integer('last_updated_experiment_n'),
  updated_at: timestamp('updated_at').notNull().defaultNow(),
}, (table) => ({
  uniq: unique().on(table.run_id, table.parameter),
}));
text


---

## 📄 File 6: `specs/_global/geometry-parameter-registry.md`

```markdown
# AeroLoop — Wing Geometry Parameter Registry
# Spec ID: GLOBAL-GEOM-001
# Status: APPROVED
# This file defines ALL mutable parameters in wing.geo.
# These bounds are HARD CONSTRAINTS — enforced by PhysicsBoundsChecker.
# Any mutation violating these bounds is REJECTED before meshing.

---

## PLANFORM PARAMETERS

| Parameter ID              | Display Name          | Unit | Min    | Max    | Default | Step_Fine | Step_Med | Step_Global |
|---------------------------|-----------------------|------|--------|--------|---------|-----------|----------|-------------|
| span_m                    | Wing Span             | m    | 10.0   | 80.0   | 35.0    | 0.25      | 1.0      | 3.0         |
| semi_span_m               | Semi-Span             | m    | 5.0    | 40.0   | 17.5    | 0.125     | 0.5      | 1.5         |
| aspect_ratio              | Aspect Ratio          | -    | 5.0    | 18.0   | 9.5     | 0.1       | 0.5      | 1.5         |
| taper_ratio               | Taper Ratio           | -    | 0.15   | 0.6    | 0.35    | 0.01      | 0.03     | 0.08        |
| sweep_quarter_chord_deg   | Quarter-Chord Sweep   | deg  | 5.0    | 50.0   | 28.0    | 0.5       | 2.0      | 6.0         |
| sweep_leading_edge_deg    | Leading Edge Sweep    | deg  | 8.0    | 60.0   | 33.0    | 0.5       | 2.0      | 6.0         |
| dihedral_deg              | Dihedral Angle        | deg  | -5.0   | 12.0   | 5.0     | 0.2       | 1.0      | 3.0         |
| root_chord_m              | Root Chord            | m    | 3.0    | 12.0   | 7.5     | 0.1       | 0.3      | 1.0         |
| tip_chord_m               | Tip Chord             | m    | 0.5    | 4.0    | 2.0     | 0.05      | 0.15     | 0.5         |
| mac_m                     | Mean Aerodynamic Chord| m    | 2.0    | 9.0    | 5.0     | 0.05      | 0.2      | 0.6         |

## TWIST PARAMETERS (spanwise, at 5 stations: root, 25%, 50%, 75%, tip)

| Parameter ID              | Display Name          | Unit | Min    | Max    | Default | Step_Fine | Step_Med | Step_Global |
|---------------------------|-----------------------|------|--------|--------|---------|-----------|----------|-------------|
| twist_root_deg            | Twist at Root         | deg  | -3.0   | 5.0    | 2.0     | 0.1       | 0.5      | 1.5         |
| twist_25pct_deg           | Twist at 25% Span     | deg  | -4.0   | 4.0    | 1.0     | 0.1       | 0.5      | 1.5         |
| twist_50pct_deg           | Twist at 50% Span     | deg  | -5.0   | 3.0    | 0.0     | 0.1       | 0.5      | 1.5         |
| twist_75pct_deg           | Twist at 75% Span     | deg  | -6.0   | 2.0    | -1.0    | 0.1       | 0.5      | 1.5         |
| twist_tip_deg             | Twist at Tip (washout)| deg  | -8.0   | 1.0    | -3.0    | 0.1       | 0.5      | 1.5         |

## THICKNESS PARAMETERS (t/c ratio, at 5 span stations)

| Parameter ID              | Display Name          | Unit | Min    | Max    | Default | Step_Fine | Step_Med | Step_Global |
|---------------------------|-----------------------|------|--------|--------|---------|-----------|----------|-------------|
| tc_root                   | t/c at Root           | -    | 0.10   | 0.22   | 0.15    | 0.003     | 0.01     | 0.03        |
| tc_25pct                  | t/c at 25% Span       | -    | 0.09   | 0.20   | 0.13    | 0.003     | 0.01     | 0.03        |
| tc_50pct                  | t/c at 50% Span       | -    | 0.08   | 0.18   | 0.12    | 0.003     | 0.01     | 0.03        |
| tc_75pct                  | t/c at 75% Span       | -    | 0.07   | 0.16   | 0.11    | 0.003     | 0.01     | 0.03        |
| tc_tip                    | t/c at Tip            | -    | 0.06   | 0.14   | 0.10    | 0.002     | 0.008    | 0.02        |

## CAMBER PARAMETERS (max camber as % chord, at 5 span stations)

| Parameter ID              | Display Name          | Unit | Min    | Max    | Default | Step_Fine | Step_Med | Step_Global |
|---------------------------|-----------------------|------|--------|--------|---------|-----------|----------|-------------|
| camber_root_pct           | Max Camber at Root    | %c   | 0.0    | 4.5    | 2.5     | 0.05      | 0.2      | 0.6         |
| camber_25pct_pct          | Max Camber at 25%     | %c   | 0.0    | 4.0    | 2.2     | 0.05      | 0.2      | 0.6         |
| camber_50pct_pct          | Max Camber at 50%     | %c   | 0.0    | 3.5    | 2.0     | 0.05      | 0.2      | 0.6         |
| camber_75pct_pct          | Max Camber at 75%     | %c   | 0.0    | 3.0    | 1.8     | 0.05      | 0.2      | 0.6         |
| camber_tip_pct            | Max Camber at Tip     | %c   | 0.0    | 2.5    | 1.5     | 0.05      | 0.2      | 0.6         |

## LEADING EDGE PARAMETERS

| Parameter ID              | Display Name          | Unit | Min    | Max    | Default | Step_Fine | Step_Med | Step_Global |
|---------------------------|-----------------------|------|--------|--------|---------|-----------|----------|-------------|
| le_radius_root_pct        | LE Radius Root        | %c   | 0.5    | 3.5    | 1.8     | 0.05      | 0.15     | 0.4         |
| le_radius_25pct_pct       | LE Radius at 25%      | %c   | 0.4    | 3.0    | 1.5     | 0.05      | 0.15     | 0.4         |
| le_radius_50pct_pct       | LE Radius at 50%      | %c   | 0.3    | 2.5    | 1.3     | 0.05      | 0.15     | 0.4         |
| le_radius_75pct_pct       | LE Radius at 75%      | %c   | 0.2    | 2.0    | 1.0     | 0.04      | 0.12     | 0.3         |
| le_radius_tip_pct         | LE Radius at Tip      | %c   | 0.15   | 1.5    | 0.8     | 0.03      | 0.1      | 0.25        |

## WINGLET PARAMETERS (activated only with topology: WINGLET gene)

| Parameter ID              | Display Name          | Unit | Min    | Max    | Default | Step_Fine | Step_Med | Step_Global |
|---------------------------|-----------------------|------|--------|--------|---------|-----------|----------|-------------|
| winglet_present           | Winglet Present       | bool | 0      | 1      | 0       | -         | -        | -           |
| winglet_height_pct_span   | Winglet Height        | %span| 2.0    | 12.0   | 5.0     | 0.2       | 0.5      | 1.5         |
| winglet_cant_deg          | Winglet Cant Angle    | deg  | 15.0   | 75.0   | 50.0    | 1.0       | 3.0      | 8.0         |
| winglet_sweep_deg         | Winglet Sweep         | deg  | 20.0   | 65.0   | 45.0    | 1.0       | 3.0      | 8.0         |
| winglet_taper_ratio       | Winglet Taper         | -    | 0.15   | 0.6    | 0.3     | 0.02      | 0.06     | 0.15        |
| winglet_toe_deg           | Winglet Toe Angle     | deg  | -3.0   | 3.0    | 0.0     | 0.2       | 0.5      | 1.0         |

---

## KNOWN PARAMETER INTERACTIONS

These interactions are pre-seeded into the knowledge graph.
The KnowledgeGraphAgent discovers and adds more during the run.

```yaml
interactions:
  - parameters: [sweep_quarter_chord_deg, tc_root]
    interaction_type: coupled
    effect: "Increasing sweep requires reducing t/c to maintain
             acceptable wave drag at transonic Mach. Decoupled designs
             (high sweep + high t/c) exhibit strong buffet penalty."
    strength: 0.85

  - parameters: [twist_tip_deg, tc_tip]
    interaction_type: coupled
    effect: "Increasing washout (more negative twist_tip) reduces
             effective angle of attack at tip. With thin tip (low t/c),
             this reduces wave drag significantly. With thick tip,
             the interaction is weaker."
    strength: 0.72

  - parameters: [aspect_ratio, sweep_quarter_chord_deg]
    interaction_type: trade_off
    effect: "High AR increases induced drag benefit but amplifies
             aeroelastic twist under load. High sweep reduces wave drag
             but increases structural weight. Their product (structural
             parameter) governs the feasibility boundary."
    strength: 0.90

  - parameters: [le_radius_root_pct, buffet_onset_cl]
    interaction_type: strong_positive
    effect: "Larger LE radius at root delays flow separation and
             improves buffet onset CL. But increases LE wave drag.
             Optimal LE radius trades these: typically 1.5-2.5%c at root."
    strength: 0.78

  - parameters: [winglet_cant_deg, span_efficiency]
    interaction_type: nonlinear
    effect: "Cant angle optimum is typically 50-65 deg for minimum
             induced drag. Below 30 deg: winglet behaves like span
             extension (good). Above 70 deg: winglet behaves like
             fuselage fairing (little drag benefit). Nonlinear optimum."
    strength: 0.82
MESH SENSITIVITY BY PARAMETER
Parameters with HIGH mesh sensitivity require remesh quality check:

Parameter	Mesh Sensitivity	Reason
le_radius_*_pct	HIGH	LE curvature governs y+ near stagnation
tc_*	MEDIUM-HIGH	t/c affects boundary layer growth rate
sweep_*_deg	MEDIUM	Affects spanwise mesh skewness
winglet_cant_deg	HIGH	Winglet junction requires fine mesh
twist_*_deg	LOW	Minimal effect on mesh topology
camber_*_pct	LOW	Smooth camber changes: minimal mesh effect
taper_ratio	MEDIUM	Affects tip chord cell count
text


---

## 📄 File 7: `specs/stage-0-initialization/requirements.md`

```markdown
# Stage 0: Initialization & Configuration
# Spec ID: STAGE-0-REQ-001
# Status: APPROVED
# Implementing: InitializationAgent, BaselineValidationAgent
# Max Duration: 30 minutes
# Gate: OPERATOR-GATE-0 (operator confirms config before loop starts)

---

## BUSINESS CONTEXT

Stage 0 is the setup phase. It establishes all invariants that the
autoresearch loop relies on. A wrong baseline, wrong metric, or wrong
operating conditions discovered at experiment 400 wastes 57 hours of
GPU time. Therefore, Stage 0 validates everything rigorously before
the first mutation is attempted.

The output of Stage 0 is a complete, immutable ExperimentRun record
and a verified, meshed, solved baseline wing — the benchmark every
subsequent experiment is compared against.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-0-001: InitializationAgent MUST create an ExperimentRun record with:
  - Unique run_id (UUID v4)
  - run_name (operator-provided, ≥ 5 chars, unique)
  - Locked metric_definition (weights must sum to 1.0 ± 0.001)
  - Locked operating_conditions (Mach, Re, altitude, cruise CL target)
  - Locked solver_config (solver type, convergence criteria, mesh level)
  All fields are IMMUTABLE after Stage 0 completes.
  Any attempt to modify them mid-run: BLOCKED by PreToolUse hook.
:::END:::

:::STRICT_REQUIREMENT:::
FR-0-002: BaselineValidationAgent MUST run the FULL experiment pipeline
on the initial wing.geo (no mutation):
  1. PhysicsBoundsChecker: initial geometry must pass all bounds
  2. GMSH: mesh at configured mesh level (default: medium, ~2M cells)
  3. MeshQualityChecker: must pass all quality criteria (RULE-031)
  4. SU2 solve: must converge (≥ 4 orders residual drop) within kill timer
  5. PolarsExtractor: must extract valid L/D, Cd, Cm, buffet_margin
  6. CompositeMetric: baseline metric M0 computed and stored as M_best
  If ANY step fails: Stage 0 FAILS. Operator must fix geometry before retry.
  Do NOT start the loop on an unvalidated baseline.
:::END:::

:::STRICT_REQUIREMENT:::
FR-0-003: Geometry parameter extraction MUST complete before loop start.
  InitializationAgent reads wing.geo and extracts ALL parameter values
  into the PostgreSQL geometry_parameters table.
  Every parameter in geometry-parameter-registry.md MUST have a value.
  Parameters not explicitly set in wing.geo: use default_value.
  This extraction seeds the knowledge graph with the initial state.
:::END:::

:::STRICT_REQUIREMENT:::
FR-0-004: Git repository initialization:
  1. Create run branch: git checkout -b run/{run_id}
  2. Baseline commit: git commit -m "exp-0: baseline M0={M0:.4f} [BASELINE]"
     with full baseline EvaluationResult in commit metadata
  3. Tag: git tag baseline-{run_name}
  4. Copy wing.geo to S3: runs/{run_id}/geometry/baseline.geo
  5. Create .aeroloop/ directory with:
     - run_config.json (immutable run configuration)
     - best_metric.json (initialized to M0)
     - current_genome.json (initialized to DEFAULT_GENOME)
     - knowledge_graph_summary.md (empty, will be populated)
  All of these must complete before OPERATOR-GATE-0.
:::END:::

:::STRICT_REQUIREMENT:::
FR-0-005: Operator Gate-0 payload MUST include:
  - Baseline metric value M0 (with breakdown: L/D, buffet_margin, wave_drag_frac)
  - Baseline operating conditions confirmation
  - Parameter count (how many geometry parameters will be optimized)
  - Estimated experiments per hour (based on baseline solve time)
  - Estimated overnight yield (experiments in 14 hours)
  - Mesh cell count and quality metrics
  - Solver convergence details from baseline run
  - Storage estimate (GB per experiment × target experiments)
  Operator must explicitly confirm before loop starts.
:::END:::

FR-0-006: If the operator requests swarm mode (SWARM_ENABLED=true):
  InitializationAgent MUST:
  1. Register this run with SwarmCoordinator
  2. Partition the design space into SWARM_TOTAL_NODES non-overlapping regions
  3. Assign this node its region (stored in .aeroloop/swarm_region.json)
  4. Verify all other nodes are registered before allowing any node to start
  Design space partitioning strategy: divide parameter space into N
  equal-volume hypercubes (N = SWARM_TOTAL_NODES).

FR-0-007: KnowledgeGraph seeding with known interactions:
  Load all pre-defined interactions from geometry-parameter-registry.md
  into the ParameterInteraction table. These are the prior knowledge —
  they weight the initial mutation proposals before any experiments run.

---

## ACCEPTANCE CRITERIA

```yaml
tests:
  - id: AC-0-001
    name: "Baseline solve completes within kill timer"
    expected:
      solve_wall_time_seconds: "< 480"  # 8 minutes
      converged: true
      residual_drop_orders: ">= 4"

  - id: AC-0-002
    name: "Baseline metric computed correctly"
    input:
      cl_cd: 18.5
      buffet_margin: 0.15
      wave_drag_fraction: 0.08
      weights: {w1: 0.6, w2: 0.2, w3: 0.2}
    expected:
      composite_metric: 11.294  # 0.6*18.5 + 0.2*0.15/0.2_norm + 0.2*0.92
      # (actual formula normalization tested in unit test)

  - id: AC-0-003
    name: "All geometry parameters extracted"
    expected:
      parameters_extracted: ">= 35"  # All registry parameters
      no_missing_required_parameters: true

  - id: AC-0-004
    name: "Git baseline commit exists"
    expected:
      baseline_commit_present: true
      commit_message_format_valid: true
      run_branch_exists: true
      baseline_tag_exists: true

  - id: AC-0-005
    name: "Swarm partitioning (if enabled)"
    precondition: "SWARM_ENABLED=true AND SWARM_TOTAL_NODES=4"
    expected:
      design_space_partitioned: true
      regions_non_overlapping: true
      regions_cover_full_space: true
      all_nodes_registered: true

  - id: AC-0-006
    name: "Stage-0 total duration"
    expected:
      duration_seconds: "< 1800"  # 30 minutes
text


---

## 📄 File 8: `specs/stage-1-geometry-mutation/requirements.md`

```markdown
# Stage 1: Geometry Mutation
# Spec ID: STAGE-1-REQ-001
# Status: APPROVED
# Implementing: GeometryMutationAgent, PhysicsBoundsChecker
# Max Duration: 3 minutes per experiment
# No operator gate (fully autonomous)

---

## BUSINESS CONTEXT

Stage 1 is the most critical stage of the autoresearch loop — it is
where the AI agent's intelligence is applied. The quality of the
mutation proposal determines the quality of the optimization. A good
mutation proposal: (1) is informed by experiment history, (2) is
informed by the knowledge graph, (3) is informed by wiki articles,
(4) follows the active genome strategy, (5) avoids recently tried
dead ends, (6) changes ONE parameter by a physically meaningful amount.

The PhysicsBoundsChecker is the safety net: it prevents the agent from
ever meshing or solving a physically impossible geometry.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-1-001: CONTEXT READING (mandatory before every mutation)
GeometryMutationAgent MUST read ALL of the following before proposing:

  a) Current geometry: geometry/wing.geo (full file read)
     Extract: current value of EVERY parameter (via GeometryParser)

  b) Experiment history: git log --oneline -10
     Extract: last 10 mutations, their outcomes, metric deltas

  c) Best state: .aeroloop/best_metric.json
     Extract: best metric value, which experiment achieved it,
              which parameter values were set at best experiment

  d) Knowledge graph: GET /api/v1/runs/{run_id}/knowledge-graph/summary
     Extract: top-5 most sensitive parameters, their hot/dead zones,
              interaction effects relevant to current parameter values

  e) Active genome: .aeroloop/current_genome.json
     Extract: mutation_strategy, parameter_weights, step_size_multiplier

  f) Relevant wiki articles: wiki/aerodynamics/{topic}.md
     Read the 1–2 most relevant articles for the intended mutation area.

  Any context read that fails: log warning, use cached value (max 30s stale).
  If ALL context reads fail: LOOP_PAUSE_REQUEST (cannot propose without context).
:::END:::

:::STRICT_REQUIREMENT:::
FR-1-002: MUTATION PROPOSAL — ONE CHANGE ONLY
The proposal MUST be exactly ONE of:
  Type A — Parameter Change:
    {
      "mutation_type": "parameter_change",
      "parameter": "{WingParameter.id}",
      "old_value": {current_value},
      "new_value": {proposed_value},
      "reasoning": "{string, min 50 chars, references specific history/wiki}",
      "expected_metric_delta": {number},
      "confidence": {0.0–1.0}
    }

  Type B — Topology Change (only when genome_strategy is TOPOLOGY_*):
    {
      "mutation_type": "topology_{change_type}",
      "parameters_changed": [{parameter, old_value, new_value}],
      "reasoning": "{string, min 100 chars}",
      "expected_metric_delta": {number},
      "confidence": {0.0–1.0}
    }

  Compound mutations (multiple unrelated parameters): BLOCKED.
  No mutation: BLOCKED (agent must always propose something).
:::END:::

:::STRICT_REQUIREMENT:::
FR-1-003: ANTI-REPETITION CHECK
The proposed mutation MUST NOT duplicate any mutation in the last
10 experiments (parameter + direction — same parameter moved in same
direction by similar magnitude). Check performed by MutationDeduplicator:

  Duplication criterion:
    same parameter AND
    |new_value - recent_new_value| < 0.1 * typical_step_fine

  If duplicate detected: agent must propose a different mutation.
  Max retries: 3. If 3 consecutive duplicates: escalate mutation strategy
  (LOCAL_SEARCH_FINE → LOCAL_SEARCH_COARSE → GLOBAL_SEARCH).
:::END:::

:::STRICT_REQUIREMENT:::
FR-1-004: PHYSICS BOUNDS CHECK (PhysicsBoundsChecker)
After proposal, before writing wing.geo, run PhysicsBoundsChecker:

  Check each proposed new_value against geometry-parameter-registry.md:
    - new_value ≥ parameter.min_value (hard lower bound)
    - new_value ≤ parameter.max_value (hard upper bound)

  Additional cross-parameter checks:
    - tc_tip < tc_root (always: wing should taper in thickness)
    - |twist_tip_deg - twist_root_deg| ≤ 10.0 (no extreme twist gradients)
    - winglet_present=1 requires winglet_height_pct_span > 0.0
    - sweep_leading_edge_deg > sweep_quarter_chord_deg (geometry constraint)

  GitNexus detect_impact call:
    For each changed parameter, call detect_impact:
    → returns: {mesh_quality_risk: "low|medium|high",
                known_interactions: ParameterInteraction[],
                historical_caution: string | null}
    → if mesh_quality_risk == "high": agent is WARNED (not blocked)
      agent must acknowledge the warning in its reasoning

  If bounds violated: RETURN bounds_violated=true, no wing.geo write.
  Agent must propose a new mutation within bounds.
:::END:::

:::STRICT_REQUIREMENT:::
FR-1-005: WING.GEO WRITE (only after bounds check passes)
GeometryMutationAgent writes the new parameter value to wing.geo.
Wing.geo format is OpenVSP CSV or GMSH .geo — both supported.
The write is atomic: write to wing.geo.tmp, then rename to wing.geo.
On any write failure: log error, keep original wing.geo, LOOP_PAUSE_REQUEST.

Record SHA256 of new wing.geo immediately after write.
This SHA256 becomes experiment.wing_geo_sha256_after.
:::END:::

FR-1-006: DETECT_IMPACT INTEGRATION
For every mutation, the knowledge graph detect_impact call MUST complete
before wing.geo is written. The PreToolUse hook enforces this:
  if tool == "write_file" AND file == "geometry/wing.geo":
    check that detect_impact has been called for this mutation
    check that detect_impact result is in current agent context
    if not: BLOCK the write

FR-1-007: GENOME STRATEGY COMPLIANCE
The mutation must follow the active genome strategy:
  LOCAL_SEARCH_FINE: step ≤ parameter.step_fine * genome.step_size_multiplier
  LOCAL_SEARCH_COARSE: step ≤ parameter.step_medium * genome.step_size_multiplier
  GLOBAL_SEARCH: step ≤ parameter.step_global * genome.step_size_multiplier
  SENSITIVITY_GUIDED: select parameter by knowledge_graph.mean_delta_metric rank
  TOPOLOGY_*: change topology-specific parameters only

Agent must cite which genome strategy it is following in its reasoning.

---

## ACCEPTANCE CRITERIA

```yaml
tests:
  - id: AC-1-001
    name: "Context reading completes within time budget"
    expected:
      context_read_duration_seconds: "< 60"
      all_context_sources_read: true

  - id: AC-1-002
    name: "Single mutation proposal format"
    expected:
      mutation_count: 1
      proposal_schema_valid: true
      reasoning_min_length: 50
      confidence_in_range: true  # 0.0–1.0

  - id: AC-1-003
    name: "Physics bounds enforcement"
    input:
      parameter: sweep_quarter_chord_deg
      new_value: 55.0     # > max of 50.0
    expected:
      bounds_check_passed: false
      wing_geo_written: false

  - id: AC-1-004
    name: "detect_impact called before write"
    expected:
      detect_impact_called: true
      detect_impact_result_in_context: true

  - id: AC-1-005
    name: "Anti-repetition: no duplicate in last 10"
    setup:
      last_10_mutations: [{parameter: "twist_tip_deg", new_value: -3.5}]
    input:
      proposed: {parameter: "twist_tip_deg", new_value: -3.48}
    expected:
      duplicate_detected: true
      alternative_proposed: true

  - id: AC-1-006
    name: "Wing.geo SHA256 recorded after write"
    expected:
      sha256_recorded: true
      sha256_matches_file: true

  - id: AC-1-007
    name: "Stage-1 total duration"
    expected:
      duration_seconds: "< 180"  # 3 minutes
text


---

## 📄 File 9: `specs/stage-2-mesh-generation/requirements.md`

```markdown
# Stage 2: Mesh Generation
# Spec ID: STAGE-2-REQ-001
# Status: APPROVED
# Implementing: MeshGeneratorAgent, MeshQualityAgent
# Max Duration: 5 minutes per experiment
# No operator gate (fully autonomous)

---

## BUSINESS CONTEXT

Mesh generation is the bridge between geometry and CFD. A bad mesh —
high skewness, poor boundary layer resolution, incorrect y+ — produces
unreliable results that contaminate the keep/revert decision. Stage 2
validates mesh quality rigorously before any compute time is spent on
a CFD solve. A failed mesh is cheaper than a failed solve.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-2-001: GMSH PIPELINE
MeshGeneratorAgent MUST execute auto_mesh.py with the current wing.geo.
The pipeline:
  1. Read wing.geo → extract all parameter values
  2. Select mesh template based on solver_config.mesh_level:
       coarse: ~500k cells, y+ target 5
       medium: ~2M cells, y+ target 2 (default)
       fine: ~8M cells, y+ target 1
  3. Set GMSH mesh size fields based on:
       - Local curvature (finer near LE, TE, winglet junction)
       - Boundary layer progression: first_cell_height from y+ target
         first_cell_height = y_plus_target * mu / (rho * u_tau)
         where u_tau estimated from flat plate correlation
       - Far-field: coarse (1/4 chord cell size at 50 chord lengths)
  4. Execute GMSH Python API (not CLI — API gives programmatic control)
  5. Export: mesh_{exp_n}.su2 (SU2 format) or mesh_{exp_n}.msh (OpenFOAM)
  6. Upload mesh to S3: runs/{run_id}/meshes/mesh_{exp_n}.su2

  Kill timer: 5 minutes. If GMSH exceeds this: MESH_FAILED (timeout).
:::END:::

:::STRICT_REQUIREMENT:::
FR-2-002: MESH QUALITY CHECK (mandatory, blocks solver start)
MeshQualityAgent MUST run checks after GMSH completes.
ALL criteria must pass before MESH_COMPLETE is sent to CFDSolverAgent.

  Criterion 1 — Orthogonality (ALL cells):
    minimum_face_orthogonality_deg > 20.0
    (computed via checkMesh for OpenFOAM; SU2: custom checker)

  Criterion 2 — Non-orthogonality:
    maximum_non_orthogonality_deg < 85.0
    average_non_orthogonality_deg < 35.0

  Criterion 3 — Skewness:
    maximum_cell_skewness < 4.0

  Criterion 4 — y+ validation (wall-bounded regions):
    y_plus_first_cell_avg: within ±50% of target
    y_plus_first_cell_max < 5 * target_y_plus

  Criterion 5 — Cell count:
    actual_cell_count: within ±20% of target for mesh_level
    coarse target: 500000 ± 100000
    medium target: 2000000 ± 400000
    fine target: 8000000 ± 1600000

  Criterion 6 — Aspect ratio (boundary layer cells):
    max_aspect_ratio_bl < 1000 (wall-adjacent cells only)

  If ANY criterion fails: MESH_FAILED (quality). Do NOT solve.
  Log: which criterion failed, actual value vs. threshold.
:::END:::

:::STRICT_REQUIREMENT:::
FR-2-003: BOUNDARY CONDITION MAPPING
MeshGeneratorAgent MUST verify that the following boundary patches
exist in the generated mesh and are correctly tagged:
  SU2 boundaries required:
    - "farfield" (type: Riemann, far-field boundary condition)
    - "wing_surface" (type: Euler wall or viscous wall)
    - "symmetry_plane" (type: Symmetry, for half-span models)
    - "wake_cut" (type: Periodic, if wake structured mesh used)
  If any boundary patch is missing: MESH_FAILED (boundary_mapping).
:::END:::

FR-2-004: MESH ARCHIVING
After quality check passes:
  1. Compute SHA256 of mesh file
  2. Upload mesh to S3 (runs/{run_id}/meshes/mesh_{exp_n}.su2)
  3. Verify SHA256 of S3 copy
  4. Record in experiment.mesh_result: {
       cell_count, y_plus_avg, quality_metrics, sha256,
       s3_key, gmsh_wall_time_seconds
     }
  5. Clean local disk: remove mesh file after S3 upload confirmed.

FR-2-005: PARAMETRIC MESH SIZING
The GMSH mesh size field MUST respond to the geometry parameters.
Specifically, mesh refinement MUST be:
  - Finer when: le_radius_* is small (high curvature needs finer mesh)
  - Finer when: winglet_present=1 (winglet junction is complex)
  - Finer when: tc_* is small (thin sections need finer wake mesh)
  - Standard otherwise: follow template targets
  This is implemented via GMSH SizeMax/SizeMin fields on geometric entities.

---

## `mesh/auto_mesh.py` — Complete Implementation Spec

```python
"""
auto_mesh.py — AeroLoop GMSH Meshing Pipeline
Spec: STAGE-2-REQ-001
Called by: MeshGeneratorAgent
Input: geometry/wing.geo, mesh level (coarse/medium/fine), experiment_n
Output: mesh_{experiment_n}.su2

MUST NOT be modified during a run (READ-ONLY except by maintainers).
"""

import gmsh
import sys
import json
import hashlib
import argparse
from pathlib import Path
from typing import Literal

# ── CONSTANTS (from packages/shared/src/constants) ──────────────────────
MESH_TARGETS = {
    "coarse": {"cells": 500_000, "y_plus": 5.0, "growth_rate": 1.3},
    "medium": {"cells": 2_000_000, "y_plus": 2.0, "growth_rate": 1.2},
    "fine":   {"cells": 8_000_000, "y_plus": 1.0, "growth_rate": 1.15},
}

QUALITY_THRESHOLDS = {
    "min_orthogonality_deg": 20.0,
    "max_non_orthogonality_deg": 85.0,
    "avg_non_orthogonality_deg": 35.0,
    "max_skewness": 4.0,
    "max_aspect_ratio_bl": 1000.0,
    "cell_count_tolerance_pct": 0.20,
}

# ── MAIN PIPELINE ────────────────────────────────────────────────────────

def generate_mesh(
    geo_file: Path,
    mesh_level: Literal["coarse", "medium", "fine"],
    experiment_n: int,
    operating_conditions: dict,
    output_dir: Path,
) -> dict:
    """
    Generate SU2-compatible mesh from wing.geo.
    Returns: mesh metadata dict (cell_count, quality_metrics, s3_key, sha256)
    Raises: MeshGenerationError on any failure.
    """

    target = MESH_TARGETS[mesh_level]
    output_file = output_dir / f"mesh_{experiment_n}.su2"

    # 1. Initialize GMSH
    gmsh.initialize()
    gmsh.model.add(f"wing_exp_{experiment_n}")
    gmsh.option.setNumber("General.Terminal", 0)  # Suppress output

    try:
        # 2. Load geometry
        _load_geometry(geo_file)

        # 3. Compute first cell height from y+ target
        first_cell_height = _compute_first_cell_height(
            y_plus=target["y_plus"],
            mach=operating_conditions["mach"],
            altitude_m=operating_conditions["altitude_m"],
        )

        # 4. Set mesh size fields
        _set_size_fields(
            target=target,
            first_cell_height=first_cell_height,
            geo_file=geo_file,
        )

        # 5. Set boundary layer fields
        _set_boundary_layer(
            first_cell_height=first_cell_height,
            growth_rate=target["growth_rate"],
            n_layers=_compute_bl_layers(target["cells"]),
        )

        # 6. Generate 3D mesh
        gmsh.model.mesh.generate(3)

        # 7. Optimize mesh
        gmsh.model.mesh.optimize("Netgen", force=True)

        # 8. Verify boundary patches
        _verify_boundary_patches()

        # 9. Export to SU2 format
        gmsh.write(str(output_file))

    finally:
        gmsh.finalize()

    # 10. Compute quality metrics
    quality = _compute_quality_metrics(output_file)

    # 11. Validate quality
    _validate_quality(quality, QUALITY_THRESHOLDS, target)

    # 12. Compute SHA256
    sha256 = _compute_sha256(output_file)

    return {
        "mesh_file": str(output_file),
        "sha256": sha256,
        "cell_count": quality["cell_count"],
        "quality_metrics": quality,
        "mesh_level": mesh_level,
        "y_plus_target": target["y_plus"],
        "first_cell_height_m": first_cell_height,
    }


def _compute_first_cell_height(y_plus: float, mach: float, altitude_m: float) -> float:
    """
    Compute first cell height for target y+.
    Uses flat plate correlation for u_tau estimate.
    Accurate to ~20% for turbulent boundary layers.
    """
    # ISA atmosphere at altitude
    rho, mu = _isa_atmosphere(altitude_m)
    a = 340.3 * (1 - 2.25577e-5 * altitude_m) ** 2.5  # Speed of sound (approx)
    u_inf = mach * a

    # Flat plate skin friction correlation (Schlichting)
    # Re_L based on MAC
    mac = 5.0  # Default MAC — will be overridden by wing.geo parser
    re_l = rho * u_inf * mac / mu
    cf = 0.026 / (re_l ** (1.0 / 7.0))  # Turbulent flat plate

    u_tau = u_inf * (cf / 2.0) ** 0.5
    y_first = y_plus * mu / (rho * u_tau)

    return y_first


def _compute_sha256(file_path: Path) -> str:
    sha256 = hashlib.sha256()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(65536), b""):
            sha256.update(chunk)
    return sha256.hexdigest()
ACCEPTANCE CRITERIA
YAML

tests:
  - id: AC-2-001
    name: "Mesh generation completes within time budget"
    expected:
      duration_seconds: "< 300"   # 5 minutes

  - id: AC-2-002
    name: "Cell count within ±20% of target"
    input:
      mesh_level: "medium"
    expected:
      cell_count:
        min: 1600000
        max: 2400000

  - id: AC-2-003
    name: "All quality criteria pass"
    expected:
      min_orthogonality_deg: "> 20.0"
      max_non_orthogonality_deg: "< 85.0"
      max_skewness: "< 4.0"
      max_aspect_ratio_bl: "< 1000.0"

  - id: AC-2-004
    name: "Required boundary patches present"
    expected:
      patches_present: ["farfield", "wing_surface", "symmetry_plane"]

  - id: AC-2-005
    name: "SHA256 computed and verified vs S3"
    expected:
      sha256_computed: true
      s3_sha256_match: true

  - id: AC-2-006
    name: "Bad geometry triggers MESH_FAILED (quality)"
    input:
      wing_geo: "test/fixtures/degenerate_wing.geo"
    expected:
      status: "failed_mesh_quality"
      solve_not_started: true
text


---

## 📄 File 10: `specs/stage-3-cfd-solve/requirements.md`

```markdown
# Stage 3: CFD Solve
# Spec ID: STAGE-3-REQ-001
# Status: APPROVED
# Implementing: CFDSolverAgent
# Max Duration: 8 minutes + 1 minute overhead = 9 minutes hard limit
# No operator gate (fully autonomous)

---

## BUSINESS CONTEXT

The CFD solve is the most expensive step in each experiment. On an
A100 with 8 MPI processes, SU2 RANS solves a 2M-cell transonic wing
in 6–8 minutes. The kill timer at 8 minutes is INVIOLABLE — it ensures
experiments are comparable and prevents runaway solves from blocking
the queue. A killed solve is recorded as TIMEOUT and reverted. This
is acceptable — not every experiment needs to converge.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-3-001: KILL TIMER — DUAL LAYER ENFORCEMENT
The kill timer MUST be enforced at TWO independent levels:

  Level 1 — OS level (run_su2.sh):
    timeout {CFD_MAX_WALL_TIME_MINUTES}m mpirun -n {SU2_MPI_PROCESSES} \
      {SU2_BINARY} {config_file}
    exit code 124 = timeout triggered

  Level 2 — Application level (ProcessManager):
    BullMQ job timeout: (CFD_MAX_WALL_TIME_MINUTES + 1) * 60 * 1000 ms
    On BullMQ timeout: send SIGKILL to solver process + all children
    Record: SOLVE_TIMEOUT in ExperimentRegistry

  If EITHER level triggers: solver is dead, resources cleaned up,
  experiment recorded as TIMEOUT, wing.geo reverted.
  The two levels are independent — either can trigger without the other.
:::END:::

:::STRICT_REQUIREMENT:::
FR-3-002: SU2 CONFIGURATION GENERATION
CFDSolverAgent MUST generate the SU2 config from the template at
solver/su2_config_template.cfg with experiment-specific values:

  Required fields (generated per experiment):
    MESH_FILENAME= {s3_downloaded_mesh_path}
    MACH_NUMBER= {operating_conditions.mach}
    AOA= {operating_conditions.alpha_cruise_deg}
    REYNOLDS_NUMBER= {operating_conditions.reynolds}
    REYNOLDS_LENGTH= {geometry.mac_m}
    KIND_TURB_MODEL= {solver_config.turbulence_model}  # Default: SST
    CONV_CAUCHY_EPS= {solver_config.convergence_eps}   # Default: 1E-10
    ITER= {CFD_MAX_ITERATIONS}
    SURFACE_FILENAME= surface_{experiment_n}
    CONV_FILENAME= history_{experiment_n}

  The config file is written to CFD_SCRATCH_DIR/{experiment_n}/su2.cfg
  The mesh file is downloaded from S3 to CFD_SCRATCH_DIR/{experiment_n}/mesh.su2
  before the solve starts.
:::END:::

:::STRICT_REQUIREMENT:::
FR-3-003: CONVERGENCE VALIDATION
After solve completes (within time budget), CFDSolverAgent MUST check
convergence from the history_{experiment_n}.csv file:

  Criterion 1 — Residual drop:
    final_residual ≤ initial_residual * 10^(-CFD_CONVERGENCE_ORDERS)
    where CFD_CONVERGENCE_ORDERS = 4 (default)

  Criterion 2 — Force coefficient stabilization:
    |CD[-1] - CD[-100]| / CD[-1] < 0.005  (CD stable in last 100 iters)
    |CL[-1] - CL[-100]| / CL[-1] < 0.003  (CL stable in last 100 iters)

  If BOTH criteria pass: converged = true
  If EITHER fails: converged = false → SOLVE_UNCONVERGED → revert
  UNCONVERGED results MUST NOT be used for keep/revert decisions.
:::END:::

:::STRICT_REQUIREMENT:::
FR-3-004: OUTPUT FILE HANDLING
After a converged solve:
  1. surface_{exp_n}.csv: upload to S3 (runs/{run_id}/cfd/surface_{exp_n}.csv)
  2. history_{exp_n}.csv: upload to S3 (runs/{run_id}/cfd/history_{exp_n}.csv)
  3. Compute SHA256 for both files before upload
  4. Verify SHA256 of S3 copies
  5. Record both S3 keys and SHA256 hashes in SolveResult
  6. CLEAN local scratch directory (CFD_SCRATCH_DIR/{experiment_n}/)
     Reason: GPU nodes have limited local storage. Clean immediately.
  If S3 upload fails: RETRY 3 times. If all fail: PIPELINE HALT.
  Surface file SHA256 becomes the ExperimentRegistry anchor for this result.
:::END:::

:::STRICT_REQUIREMENT:::
FR-3-005: MPI RESOURCE MANAGEMENT
  - Only ONE SU2 solve at a time per node (Redis exclusive lock)
  - Lock key: "aeroloop:solve_lock:{node_id}"
  - Lock acquired before spawning mpirun
  - Lock released after solver terminates AND scratch dir cleaned
  - If lock cannot be acquired within 60 seconds: log error, wait 120s, retry
  - Never start a solve if the previous solve's lock is still held
:::END:::

FR-3-006: ALPHA SWEEP (optional, when buffet_onset_cl is needed)
When solver_config.run_alpha_sweep = true:
  Run SU2 at alpha: [alpha_cruise - 2°, alpha_cruise, alpha_cruise + 2°,
                     alpha_cruise + 4°, alpha_cruise + 6°]
  Purpose: determine CL_max, buffet onset CL (d(CL)/d(alpha) drops below threshold)
  Alpha sweep increases total solve time. Kill timer applies to EACH alpha point.
  If any alpha point times out: mark buffet_onset_cl = null (no buffet data).

---

## `solver/run_su2.sh` — Complete Implementation Spec

```bash
#!/bin/bash
# run_su2.sh — AeroLoop SU2 RANS Solver with Kill Timer
# Spec: STAGE-3-REQ-001
# Called by: CFDSolverAgent (via ProcessManager)
# Environment: GPU node with A100, 8 MPI processes
# KILL TIMER LEVEL 1: timeout command (OS-level)

set -euo pipefail

EXPERIMENT_N="${1:?experiment_n required}"
SCRATCH_DIR="${CFD_SCRATCH_DIR:?CFD_SCRATCH_DIR required}/${EXPERIMENT_N}"
MAX_TIME="${CFD_MAX_WALL_TIME_MINUTES:?CFD_MAX_WALL_TIME_MINUTES required}m"
SU2_BIN="${SU2_BINARY:?SU2_BINARY required}"
MPI_PROCS="${SU2_MPI_PROCESSES:-8}"

CONFIG_FILE="${SCRATCH_DIR}/su2.cfg"
LOG_FILE="${SCRATCH_DIR}/su2_stdout.log"
ERR_FILE="${SCRATCH_DIR}/su2_stderr.log"
RESULT_FILE="${SCRATCH_DIR}/solve_result.json"

echo "AeroLoop SU2 Solver | exp=${EXPERIMENT_N} | procs=${MPI_PROCS} | timer=${MAX_TIME}"
echo "Config: ${CONFIG_FILE}"

START_TIME=$(date +%s)

# ── KILL TIMER LEVEL 1: OS timeout ──────────────────────────────────────
timeout "${MAX_TIME}" \
  mpirun -n "${MPI_PROCS}" \
    --bind-to socket \
    --map-by socket \
    "${SU2_BIN}" "${CONFIG_FILE}" \
  > "${LOG_FILE}" 2> "${ERR_FILE}"

EXIT_CODE=$?
END_TIME=$(date +%s)
WALL_TIME=$((END_TIME - START_TIME))

# ── EXIT CODE HANDLING ──────────────────────────────────────────────────
if [ ${EXIT_CODE} -eq 124 ]; then
  echo "KILL_TIMER_TRIGGERED: wall_time=${WALL_TIME}s limit=${MAX_TIME}"
  echo '{"status":"timeout","wall_time_seconds":'"${WALL_TIME}"'}' > "${RESULT_FILE}"
  exit 124
elif [ ${EXIT_CODE} -ne 0 ]; then
  echo "SOLVER_ERROR: exit_code=${EXIT_CODE} wall_time=${WALL_TIME}s"
  echo '{"status":"error","exit_code":'"${EXIT_CODE}"',"wall_time_seconds":'"${WALL_TIME}"'}' > "${RESULT_FILE}"
  exit ${EXIT_CODE}
else
  echo "SOLVER_COMPLETE: wall_time=${WALL_TIME}s"
  echo '{"status":"complete","wall_time_seconds":'"${WALL_TIME}"'}' > "${RESULT_FILE}"
  exit 0
fi
ACCEPTANCE CRITERIA
YAML

tests:
  - id: AC-3-001
    name: "Kill timer triggers at exactly CFD_MAX_WALL_TIME_MINUTES"
    setup:
      CFD_MAX_WALL_TIME_MINUTES: 1
      solver: "infinite_loop_mock"
    expected:
      exit_code: 124
      wall_time_seconds:
        min: 55
        max: 70
      status: "failed_solve_timeout"
      wing_geo_reverted: true

  - id: AC-3-002
    name: "Convergence check correctly classifies converged solve"
    input:
      residual_history: "test/fixtures/converged_history.csv"
      convergence_orders_required: 4
    expected:
      converged: true
      residual_drop_orders: ">= 4"

  - id: AC-3-003
    name: "Convergence check correctly classifies unconverged solve"
    input:
      residual_history: "test/fixtures/oscillating_history.csv"
    expected:
      converged: false
      status: "failed_solve_unconverged"
      wing_geo_reverted: true

  - id: AC-3-004
    name: "Output files uploaded to S3 with correct SHA256"
    expected:
      surface_csv_s3_uploaded: true
      history_csv_s3_uploaded: true
      sha256_verified: true
      local_scratch_cleaned: true

  - id: AC-3-005
    name: "MPI lock prevents concurrent solves"
    expected:
      concurrent_solves_on_same_node: 0
      lock_acquisition_timeout_seconds: 60
text


---

## 📄 File 11: `specs/stage-4-results-extraction/requirements.md`

```markdown
# Stage 4: Results Extraction & Metric Computation
# Spec ID: STAGE-4-REQ-001
# Status: APPROVED
# Implementing: PolarsExtractorAgent, MetricComputerAgent
# Max Duration: 3 minutes per experiment
# No operator gate (fully autonomous)

---

## BUSINESS CONTEXT

Stage 4 transforms raw SU2 output files into the structured aerodynamic
data that drives the keep/revert decision. This is where the physics
lives — extracting not just L/D but buffet onset margin, wave drag
fraction, and ultimately the composite metric M that determines whether
the past 8 minutes of GPU time resulted in a geometry worth keeping.

Every number extracted here is anchored to a SHA256-verified source file.
The ExperimentRegistry becomes the immutable scientific record.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-4-001: SHA256 VERIFICATION BEFORE EXTRACTION
PolarsExtractorAgent MUST verify the SHA256 of the surface_{exp_n}.csv
against the value stored in SolveResult BEFORE reading any data from it.

  verification_steps:
    1. Download surface_{exp_n}.csv from S3
    2. Compute SHA256 of downloaded file
    3. Compare against SolveResult.surface_csv_sha256
    4. If mismatch: PIPELINE HALT + alert operator (data integrity violated)
    5. If match: proceed with extraction

  This check is non-negotiable. It ensures the file has not been
  corrupted in transit or tampered with.
:::END:::

:::STRICT_REQUIREMENT:::
FR-4-002: AERODYNAMIC COEFFICIENT EXTRACTION
From surface_{exp_n}.csv (SU2 surface integration output):

  Direct extractions:
    cl    = column "CL" (lift coefficient, full span integrated)
    cd    = column "CD" (total drag coefficient)
    cm    = column "CMy" (pitching moment about y-axis)
    cl_cd = cl / cd  (L/D ratio)

  Wave drag decomposition (from Computational drag breakdown):
    Method: Paparone-Tognaccini far-field method (if available in SU2 output)
    Fallback: cd_wave = cd - cd_induced - cd_viscous
      where cd_induced = cl² / (π * AR * e)  (Prandtl, e estimated from Oswald)
      and cd_viscous from SU2 skin friction integration

  Buffet onset detection (requires alpha sweep results):
    buffet_onset_cl = CL at which d(CL)/d(alpha) drops below 0.7 * (dCL/dAlpha)_linear
    buffet_margin = buffet_onset_cl - cruise_cl
    If no alpha sweep: buffet_margin = null, buffet_onset_cl = null
    Note: null buffet_margin → composite metric uses w2=0 (skip buffet term)

  All extracted values stored in EvaluationResult.
:::END:::

:::STRICT_REQUIREMENT:::
FR-4-003: COMPOSITE METRIC COMPUTATION
MetricComputerAgent computes M using the LOCKED weights from run_config.json:

  Standard formula:
    M = w1 * normalize(cl_cd) +
        w2 * normalize(buffet_margin) +
        w3 * (1 - cd_wave/cd)

  Normalization (to make terms dimensionally consistent):
    normalize(cl_cd) = cl_cd / cl_cd_baseline    (ratio vs. baseline)
    normalize(buffet_margin) = buffet_margin / 0.20  (normalize to 20% margin)

  If buffet_margin is null (no alpha sweep):
    M = w1 * normalize(cl_cd) + w3 * (1 - cd_wave/cd)
    weights renormalized: w1' = w1/(w1+w3), w3' = w3/(w1+w3)

  metric_delta_vs_best = M - best_metric (from .aeroloop/best_metric.json)
  metric_delta_vs_previous = M - previous_experiment_metric

  The composite metric value and all components are stored in
  EvaluationResult and registered in ExperimentRegistry.
:::END:::

:::STRICT_REQUIREMENT:::
FR-4-004: EXPERIMENT REGISTRY ENTRY
After metric computation, MetricComputerAgent MUST create an
ExperimentRegistry entry with:
  - experiment_n (monotonic integer, from PostgreSQL sequence)
  - composite_metric value
  - metric_delta_vs_best
  - surface_csv_sha256 (the anchor)
  - history_csv_sha256
  - all raw coefficients (cl, cd, cm, cd_wave, cd_induced, cd_viscous)
  - buffet_onset_cl, buffet_margin
  - wing_geo_sha256_after (from Stage 1)
  - mutation details (from Stage 1)
  - wall_time_seconds (from Stage 3)
  - converged: true
  This entry is APPEND-ONLY. Cannot be modified or deleted.
:::END:::

FR-4-005: SANITY CHECKS (physical reasonableness)
Before accepting extraction results, run sanity checks:
  - cl must be between 0.0 and 2.5 (for subsonic/transonic cruise configs)
  - cd must be between 0.005 and 0.10 (transport aircraft reasonable range)
  - cl_cd must be between 5.0 and 35.0
  - |cm| must be < 0.3
  - cd_wave/cd must be between 0.0 and 0.5
  If any sanity check fails: flag as SANITY_VIOLATION in EvaluationResult.
  Sanity violations are KEPT and logged — they may indicate an interesting
  configuration. But the operator is notified via Slack.

---

## `evaluate/extract_polars.py` — Implementation Spec

```python
"""
extract_polars.py — AeroLoop SU2 Results Extractor
Spec: STAGE-4-REQ-001
Input: surface_{exp_n}.csv, history_{exp_n}.csv (from S3)
Output: EvaluationResult JSON

MUST NOT be modified during a run (READ-ONLY except by maintainers).
"""

import pandas as pd
import numpy as np
import hashlib
import json
import argparse
from pathlib import Path
from dataclasses import dataclass, asdict


@dataclass
class AeroCoefficients:
    cl: float
    cd: float
    cm: float
    cl_cd: float
    cd_wave: float
    cd_induced: float
    cd_viscous: float
    buffet_onset_cl: float | None
    buffet_margin: float | None


def verify_sha256(file_path: Path, expected_sha256: str) -> None:
    """
    Verify file SHA256 matches expected value.
    RAISES IntegrityError if mismatch — pipeline halts.
    """
    sha256 = hashlib.sha256()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(65536), b""):
            sha256.update(chunk)
    computed = sha256.hexdigest()
    if computed != expected_sha256:
        raise IntegrityError(
            f"SHA256 MISMATCH: {file_path.name}\n"
            f"  Expected: {expected_sha256}\n"
            f"  Computed: {computed}\n"
            f"  ACTION: PIPELINE HALT — data integrity violated"
        )


def extract_coefficients(
    surface_csv: Path,
    history_csv: Path,
    alpha_sweep_csvs: list[Path] | None,
    operating_conditions: dict,
    geometry_params: dict,
) -> AeroCoefficients:
    """
    Extract aerodynamic coefficients from SU2 output files.
    All extractions are deterministic from the input files.
    """
    # Load surface data (final time step)
    surface = pd.read_csv(surface_csv)

    # Extract integrated forces (SU2 writes totals in header comments)
    # SU2 surface CSV format: rows are surface points; CL/CD in file header
    cl = _extract_header_value(surface_csv, "Total CL:")
    cd = _extract_header_value(surface_csv, "Total CD:")
    cm = _extract_header_value(surface_csv, "Total CMy:")
    cl_cd = cl / cd if cd > 0 else 0.0

    # Wave drag decomposition (far-field method preferred)
    cd_wave, cd_induced, cd_viscous = _decompose_drag(
        surface=surface,
        cl=cl,
        cd=cd,
        aspect_ratio=geometry_params.get("aspect_ratio", 9.5),
        mach=operating_conditions["mach"],
    )

    # Buffet onset from alpha sweep
    buffet_onset_cl, buffet_margin = None, None
    if alpha_sweep_csvs:
        buffet_onset_cl, buffet_margin = _compute_buffet_onset(
            alpha_sweep_csvs=alpha_sweep_csvs,
            cruise_cl=cl,
        )

    return AeroCoefficients(
        cl=cl, cd=cd, cm=cm, cl_cd=cl_cd,
        cd_wave=cd_wave, cd_induced=cd_induced, cd_viscous=cd_viscous,
        buffet_onset_cl=buffet_onset_cl,
        buffet_margin=buffet_margin,
    )


def compute_composite_metric(
    coeffs: AeroCoefficients,
    weights: dict,  # {w1, w2, w3}
    baseline: dict, # {cl_cd_baseline, buffet_margin_baseline}
) -> float:
    """
    Compute composite aerodynamic metric M.
    Formula: M = w1*norm(L/D) + w2*norm(buffet) + w3*(1-Cdw/Cd)
    Normalization: ratios vs. baseline values.
    """
    w1, w2, w3 = weights["w1"], weights["w2"], weights["w3"]

    term_ld = (coeffs.cl_cd / baseline["cl_cd_baseline"]) if baseline["cl_cd_baseline"] > 0 else 0
    term_wave = 1.0 - (coeffs.cd_wave / coeffs.cd) if coeffs.cd > 0 else 0

    if coeffs.buffet_margin is not None and w2 > 0:
        bm_norm = coeffs.buffet_margin / 0.20  # Normalize to 20% margin target
        total_weight = w1 + w2 + w3
        return (w1 * term_ld + w2 * bm_norm + w3 * term_wave) / total_weight
    else:
        # No buffet data: redistribute w2 to remaining terms
        total_weight = w1 + w3
        if total_weight == 0:
            return 0.0
        return (w1 * term_ld + w3 * term_wave) / total_weight
ACCEPTANCE CRITERIA
YAML

tests:
  - id: AC-4-001
    name: "SHA256 verification blocks corrupted file"
    input:
      file: "test/fixtures/surface_001.csv"
      expected_sha256: "abc123..."  # Wrong hash
    expected:
      raises: IntegrityError
      pipeline_status: "halted"

  - id: AC-4-002
    name: "Coefficient extraction matches reference values"
    input:
      surface_csv: "test/fixtures/naca0012_surface_ref.csv"
      mach: 0.75
      alpha: 2.0
    expected:
      cl:
        value: 0.512
        tolerance: 0.005
      cd:
        value: 0.0089
        tolerance: 0.0002
      cl_cd:
        value: 57.5
        tolerance: 1.0

  - id: AC-4-003
    name: "Composite metric formula correctness"
    input:
      cl_cd: 20.0
      cl_cd_baseline: 18.5
      buffet_margin: 0.18
      cd_wave: 0.0012
      cd: 0.0089
      weights: {w1: 0.6, w2: 0.2, w3: 0.2}
    expected:
      composite_metric: 1.127  # (0.6*(20/18.5) + 0.2*(0.18/0.20) + 0.2*(1-0.0012/0.0089)) / 1.0

  - id: AC-4-004
    name: "ExperimentRegistry entry created (append-only)"
    expected:
      registry_entry_created: true
      entry_immutable: true  # Verify no UPDATE possible

  - id: AC-4-005
    name: "Sanity check flags unreasonable results"
    input:
      cl: 3.5  # Too high for cruise
    expected:
      sanity_violation_flagged: true
      operator_notification_sent: true
      experiment_not_halted: true  # Sanity violations don't halt — just warn
text


---

## 📄 File 12: `specs/stage-5-keep-revert/requirements.md`

```markdown
# Stage 5: Keep/Revert Decision
# Spec ID: STAGE-5-REQ-001
# Status: APPROVED
# Implementing: KeepRevertAgent
# Max Duration: 2 minutes per experiment
# No operator gate (fully autonomous)

---

## BUSINESS CONTEXT

Stage 5 is the pivot point of the autoresearch loop — the moment where
the composite metric determines whether the last 8 minutes of compute
produced a better wing or not. The keep/revert decision is binary and
deterministic: no ambiguity, no partial keeps, no "almost good enough."
Git is the mechanism. The commit message is the experiment log.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-5-001: KEEP/REVERT DECISION RULE (deterministic, no exceptions)
  Decision := M_new > M_best

  If M_new > M_best (KEEP):
    1. git add geometry/wing.geo
    2. git commit with MANDATORY format:
       "exp-{N}: M={M_new:.4f} delta=+{delta:.4f} L/D={cl_cd:.2f} [{KEEP}]
        mutation: {parameter}={old:.4f}→{new:.4f}
        mesh: {cell_count} cells | solve: {wall_time}s | convg: {residual_orders:.1f}ord"
    3. Copy wing.geo to S3: runs/{run_id}/geometry/exp-{N}.geo
    4. Update .aeroloop/best_metric.json:
       {experiment_n, composite_metric, cl_cd, buffet_margin, wave_drag_frac,
        parameter_values_at_best (full snapshot of all geometry parameters)}
    5. Update best_experiment_n and best_metric_value in ExperimentRun record

  If M_new ≤ M_best (REVERT):
    1. git checkout geometry/wing.geo
       (restores wing.geo to last committed state — i.e., the best known state)
    2. Verify: git diff --exit-code geometry/wing.geo (must return 0)
    3. git commit is NOT made
    4. Log: "exp-{N}: M={M_new:.4f} delta={delta:.4f} [{REVERT}] reason=no_improvement"
    5. .aeroloop/best_metric.json is NOT updated

  M_best is defined as: the best composite metric achieved in THIS run.
  It is initialized to M0 (baseline) at Stage 0.
  It is ONLY updated when a KEEP decision is made.
:::END:::

:::STRICT_REQUIREMENT:::
FR-5-002: GIT WORKING TREE CLEANLINESS
Before every experiment begins, KeepRevertAgent MUST verify:
  git diff --exit-code geometry/wing.geo
  must return exit code 0 (working tree clean for wing.geo)

  If the working tree is dirty at experiment start: PIPELINE HALT.
  This indicates a failed revert from a previous experiment —
  a serious state inconsistency requiring operator inspection.
  Do NOT proceed with a dirty working tree.
:::END:::

:::STRICT_REQUIREMENT:::
FR-5-003: CONSECUTIVE FAILURE TRACKING
KeepRevertAgent maintains a consecutive_failure_counter:
  Increments on: REVERT + any failed experiment (TIMEOUT, UNCONVERGED, etc.)
  Resets on: KEEP decision

  If consecutive_failure_counter >= 3:
    1. LOOP_PAUSE_REQUEST to LoopOrchestrator
    2. Notify operator via Slack: "AeroLoop paused: 3 consecutive failures.
       Last mutation: {mutation}. Operator action required."
    3. Wait for operator acknowledgment via CLI or dashboard
    4. Operator can: resume (reset counter), change genome strategy, stop run

  This prevents the loop from spinning fruitlessly when stuck in an
  invalid geometry region or experiencing systematic solver issues.
:::END:::

:::STRICT_REQUIREMENT:::
FR-5-004: EXPERIMENT RECORD FINALIZATION
After keep or revert, KeepRevertAgent MUST update the Experiment record:
  - decision: "keep" | "revert"
  - status: "kept" | "reverted"
  - git_commit_hash: (if kept)
  - wing_geo_s3_key: (if kept)
  - completed_at: ISO timestamp
  - total_wall_time_seconds: (end_time - experiment.started_at)
  AND increment ExperimentRun:
  - experiment_count += 1
  - kept_count += 1 (if kept) OR reverted_count += 1 (if reverted)
:::END:::

FR-5-005: IMPROVEMENT RATE TRACKING
KeepRevertAgent tracks rolling improvement rate for GEP fitness scoring:
  improvement_rate = kept_count / total_count (rolling 50-experiment window)
  improvement_per_hour = delta_metric_per_kept / hours_elapsed
  These metrics are stored in PostgreSQL and used by GEPOrchestrator
  to evaluate genome fitness.

---

## ACCEPTANCE CRITERIA

```yaml
tests:
  - id: AC-5-001
    name: "KEEP decision commits to git with correct format"
    input:
      M_new: 1.145
      M_best: 1.132
      experiment_n: 47
      mutation: {parameter: "twist_tip_deg", old: -3.0, new: -3.5}
    expected:
      decision: "keep"
      git_commit_made: true
      commit_message_matches_format: true  # Regex validated
      best_metric_json_updated: true
      s3_geometry_snapshot_created: true

  - id: AC-5-002
    name: "REVERT decision restores wing.geo"
    input:
      M_new: 1.125
      M_best: 1.132
    expected:
      decision: "revert"
      git_commit_made: false
      wing_geo_restored: true  # git diff returns 0
      best_metric_json_unchanged: true

  - id: AC-5-003
    name: "Loop pauses on 3 consecutive failures"
    setup:
      simulate: 3 consecutive TIMEOUT experiments
    expected:
      loop_status: "paused"
      operator_notification_sent: true
      loop_does_not_start_exp_4: true

  - id: AC-5-004
    name: "Dirty working tree triggers PIPELINE HALT"
    setup:
      wing_geo_dirty: true  # Uncommitted change in wing.geo
    expected:
      pipeline_status: "halted"
      experiment_not_started: true

  - id: AC-5-005
    name: "Consecutive failure counter resets on KEEP"
    setup:
      consecutive_failure_counter: 2
      M_new: "better than M_best"
    expected:
      consecutive_failure_counter: 0
      loop_not_paused: true
text


---

## 📄 File 13: `specs/stage-6-wiki-compilation/requirements.md`

```markdown
# Stage 6: LLM Wiki Compilation
# Spec ID: STAGE-6-REQ-001
# Status: APPROVED
# Implementing: WikiCompilerAgent
# Trigger: every 10 experiments (PostToolUse hook)
# Max Duration: 5 minutes (background, non-blocking)
# No operator gate

---

## BUSINESS CONTEXT

The LLM Wiki is what makes AeroLoop smarter over time. Without it,
every mutation proposal starts from scratch — reading 10 git log lines
and making a decision. With it, by experiment 200, the agent has a
rich, compounding knowledge base of parameter sensitivities, interaction
effects, dead ends, and best practices — specific to THIS wing and THIS
operating condition. The wiki is what separates a random walk from an
intelligent search.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-6-001: WIKI COMPILATION TRIGGER
WikiCompilerAgent is triggered by the PostToolUse hook when:
  (experiment_n % WIKI_TRIGGER_EXPERIMENTS == 0) AND (experiment_n > 0)

  The compilation MUST run as a background job (BullMQ):
    - It does NOT block the main loop
    - The main loop continues to experiment N+1 immediately
    - Wiki compilation timeout: 5 minutes (BullMQ job timeout)
    - If timeout: log warning, skip this cycle, continue loop

  Compilation uses the batch of experiments since the LAST compilation:
    experiments_to_compile = experiments from (N-9) to N inclusive
:::END:::

:::STRICT_REQUIREMENT:::
FR-6-002: ARTICLE GENERATION PROTOCOL
For each batch of 10 experiments, WikiCompilerAgent generates:

  Article 1 — Parameter Sensitivity Article (always generated):
    File: wiki/aerodynamics/{parameter_id}-sensitivity.md
    Content:
      # Parameter: {parameter.display_name}
      ## Experiments This Batch: {list}
      ## Observed Sensitivity in This Batch
        - Improvements: {experiments where change improved metric}
        - Regressions: {experiments where change hurt metric}
        - Key insight: {LLM-generated insight from batch data}
      ## Cumulative Knowledge (all experiments to date)
        - Best value found: {value} at experiment {N}
        - Hot zones: {value ranges that consistently improve}
        - Dead zones: {value ranges that consistently hurt}
        - Optimal step size observed: {typical successful delta}
      ## Interaction Effects Observed
        - {parameter A} + {parameter B}: {observed coupling}
      ## Do Not Try
        - {specific value ranges that failed repeatedly}
    Update strategy: APPEND to existing article if it exists,
    creating cumulative knowledge. Do not overwrite.

  Article 2 — Experiment Batch Summary (always generated):
    File: wiki/experiments/batch-{N-9}-to-{N}.md
    Content:
      # Experiment Batch: {N-9} to {N}
      ## Kept: {kept_count}/{10}
      ## Best metric in batch: {value}
      ## Worst metric in batch: {value}
      ## Most effective mutation: {mutation description}
      ## Patterns observed: {LLM insight}
      ## Recommendation for next batch: {which parameters to focus on}

  Article 3 — Cross-Parameter Interaction Article (when interaction detected):
    Trigger: KnowledgeGraphAgent detected new interaction in last 10 experiments
    File: wiki/aerodynamics/interactions/{param_a}-{param_b}-coupling.md
    Content: Observed coupling, conditions, recommended strategy
:::END:::

:::STRICT_REQUIREMENT:::
FR-6-003: KNOWLEDGE GRAPH SUMMARY UPDATE
After article generation, WikiCompilerAgent MUST update:
  .aeroloop/knowledge_graph_summary.md
  Content:
    # Knowledge Graph Summary (updated: exp-{N})
    ## Top 5 Most Sensitive Parameters (by mean_delta_metric)
    | Rank | Parameter | Mean ΔMetric | Experiments | Hot Zone | Dead Zone |
    |------|-----------|-------------|-------------|----------|-----------|
    | 1    | ...       | ...         | ...         | ...      | ...       |
    ...
    ## Active Interactions
    | Parameters | Type | Strength | Recommendation |
    ...
    ## Current Best Configuration
    | Parameter | Value | vs. Baseline |
    ...
    ## Recommended Next Mutation
    Based on sensitivity ranking and current parameter values:
    "Consider {parameter} from {current} toward {hot_zone_center}"

  This file is read by GeometryMutationAgent at STEP 1 of every experiment.
:::END:::

FR-6-004: WIKI ARTICLE CROSS-LINKING
After generating new articles, WikiCompilerAgent scans all articles
for mentions of other parameters and creates markdown links:
  Any mention of a WingParameter.id → link to that parameter's sensitivity article
  Any mention of experiment number → link to batch summary article
  This creates a navigable wiki, not isolated articles.

FR-6-005: WIKI INJECTION INTO MUTATION PROMPTS
After compilation, the PostToolUse hook updates the mutation prompt context:
  GeometryMutationAgent's next invocation gets:
    "New wiki articles available: {list of new article paths}"
  The agent MUST read the 1–2 most relevant new articles before
  its next mutation proposal (enforced by program.md STEP 1e).

---

## `agent/wiki_compiler.md` — Skill Specification

```markdown
# Wiki Compiler Skill
# Triggered: every 10 experiments
# You are the WikiCompilerAgent.
# Your job: synthesize the last 10 experiments into wiki knowledge.

## INPUTS AVAILABLE TO YOU
- experiments_{N-9}_to_{N}.json — full EvaluationResult for each experiment
- current wiki articles (read-only references)
- parameter_sensitivity_db — current PostgreSQL sensitivity records

## PROTOCOL

### Step 1: Identify the dominant parameter(s) in this batch
Which parameters were mutated most? Which mutations kept vs. reverted?
Compute: keep_rate per parameter, mean_delta_metric per parameter.

### Step 2: Identify the most important insight from this batch
Look for:
  - A parameter that consistently improved metric when moved in one direction
  - A parameter that consistently hurt metric
  - An interaction: "every time we moved param A up, param B's effect changed"
  - A dead end: "we tried this range 3 times, it never works"

### Step 3: Write the Parameter Sensitivity Article
Follow the format in FR-6-002 exactly.
Use specific numbers. Reference specific experiment numbers.
DO NOT be vague. Example:
  ✅ "Increasing twist_tip from -3° to -3.5° (experiments 41, 47, 52)
      improved L/D by 0.3–0.8% without measurable buffet penalty."
  ❌ "Changing twist may improve performance."

### Step 4: Write the Batch Summary Article
What was the most effective mutation in this batch?
What should the next 10 experiments focus on?
Be specific and actionable.

### Step 5: Update knowledge_graph_summary.md
Update the top-5 sensitivity ranking with new data.
Update the "Recommended Next Mutation" section.

### Output format
JSON with keys: articles_written (list of paths), summary_updated (bool)
ACCEPTANCE CRITERIA
YAML

tests:
  - id: AC-6-001
    name: "Wiki compilation does not block main loop"
    expected:
      loop_continues_during_compilation: true
      exp_N_plus_1_starts_before_wiki_complete: true

  - id: AC-6-002
    name: "Articles generated with specific numbers"
    validation: "articles must contain specific experiment numbers and metric values"
    expected:
      experiment_references_present: true
      metric_values_present: true
      vague_generalizations_absent: true  # Validated by LLM judge

  - id: AC-6-003
    name: "knowledge_graph_summary.md updated"
    expected:
      summary_file_updated: true
      top_5_parameters_listed: true
      recommended_next_mutation_present: true

  - id: AC-6-004
    name: "Wiki compilation completes within 5 minutes"
    expected:
      duration_seconds: "< 300"

  - id: AC-6-005
    name: "Cross-links created between articles"
    expected:
      parameter_mentions_linked: true
      experiment_references_linked: true
text


---

## 📄 File 14: `specs/stage-7-gitnexus-graph/requirements.md`

```markdown
# Stage 7: GitNexus Knowledge Graph
# Spec ID: STAGE-7-REQ-001
# Status: APPROVED
# Implementing: KnowledgeGraphAgent
# Trigger: after EVERY experiment (PostToolUse hook, background)
# Max Duration: 2 minutes per update
# No operator gate

---

## BUSINESS CONTEXT

The GitNexus-inspired Knowledge Graph is the memory that makes the
GeometryMutationAgent's context reads meaningful. Without it, the agent
reads git logs and guesses. With it, the agent reads a structured,
quantitative model of parameter sensitivities, interaction effects,
and historical dead zones — specific to this optimization run.

Every experiment updates the graph. Every mutation proposal reads it.
The graph is the bridge between raw experiment results and structured
aerodynamic intelligence.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-7-001: POST-EXPERIMENT GRAPH UPDATE (every experiment)
After every experiment completes (kept or reverted), KnowledgeGraphAgent
MUST update ParameterSensitivity records for the mutated parameter:

  For a KEPT experiment (M_new > M_best):
    delta_metric = M_new - M_best (positive)
    direction = sign(new_value - old_value)
    Update ParameterSensitivity for this parameter:
      - experiments_tested += 1
      - Add data point to sensitivity distribution
      - Recompute: mean_delta_metric, std_delta_metric (rolling, all experiments)
      - If delta > max_improvement: update max_improvement and best_value_found
      - Update hot_zones: if new_value is in this zone consistently improving,
        expand or confirm the zone

  For a REVERTED experiment (M_new ≤ M_best):
    delta_metric = M_new - M_best (negative or zero)
    Update ParameterSensitivity:
      - experiments_tested += 1
      - Add negative data point to sensitivity distribution
      - Update dead_zones: if this value range consistently hurts metric,
        add/expand dead zone entry

  Update frequency: after EVERY experiment.
  Compute time target: < 60 seconds per update.
:::END:::

:::STRICT_REQUIREMENT:::
FR-7-002: DETECT_IMPACT IMPLEMENTATION
The detect_impact function is the core API of the knowledge graph.
It is called by GeometryMutationAgent before every mutation (RULE-022).

  Input:
    {
      "parameter": "{WingParameter.id}",
      "old_value": {number},
      "new_value": {number},
      "current_wing_params": {full parameter snapshot}
    }

  Output:
    {
      "mesh_quality_risk": "low" | "medium" | "high",
      "mesh_sensitivity": "LOW" | "MEDIUM" | "HIGH" (from registry),
      "known_interactions": [
        {
          "interacting_parameter": "{param_id}",
          "interaction_type": "coupled" | "trade_off" | "strong_positive" | "nonlinear",
          "effect_description": "{string}",
          "strength": {0.0–1.0},
          "source": "pre_seeded" | "observed_run"
        }
      ],
      "historical_caution": "{string | null}",
      "dead_zone_warning": {
        "is_dead_zone": boolean,
        "dead_zone_range": {min, max},
        "experiments_failed_in_zone": number
      },
      "hot_zone_indicator": {
        "is_hot_zone": boolean,
        "hot_zone_range": {min, max},
        "experiments_succeeded_in_zone": number
      },
      "similar_experiments": [
        {
          "experiment_n": number,
          "similar_parameter_value": number,
          "delta_metric": number,
          "outcome": "kept" | "reverted"
        }
      ],
      "recommendation": "{string — agent guidance}",
      "confidence": {0.0–1.0}
    }

  Performance requirement: detect_impact MUST respond in < 500ms.
  It is called in the critical path of the main loop.
:::END:::

:::STRICT_REQUIREMENT:::
FR-7-003: INTERACTION DISCOVERY
KnowledgeGraphAgent MUST detect parameter interactions from experiment data.

  Interaction discovery algorithm:
    After every 20 experiments for a parameter pair (A, B):
    1. Find experiments where parameter A was changed
    2. Split by: whether parameter B was above/below its median value
    3. Compare delta_metric distributions in each split
    4. If Mann-Whitney U test p < 0.05 (B value significantly affects A's outcome):
       Record ParameterInteractionObserved:
         {parameter_a, parameter_b, interaction_strength,
          condition: "when B > median", effect: "A's sensitivity increases/decreases"}
    5. Update both parameters' known_interactions lists

  Discovered interactions are stored in PostgreSQL AND injected into
  the detect_impact output for subsequent mutation proposals.
:::END:::

:::STRICT_REQUIREMENT:::
FR-7-004: DEAD ZONE DEFINITION AND ENFORCEMENT
A dead zone is a contiguous parameter value range where:
  - ≥ 3 experiments have been attempted
  - 0 experiments have been kept
  - All delta_metric values were negative

  Dead zone definition:
    range = [min(tried_values), max(tried_values)]
    experiments_failed = count of experiments in range
    mean_penalty = mean(delta_metric) for experiments in range

  Dead zone stored in ParameterSensitivity.dead_zones (array).
  Dead zones are INCLUDED in detect_impact output with warning.
  Dead zones are NOT hard blocks — they are warnings.
  The agent may still propose mutations in dead zones but must
  acknowledge them in its reasoning. Repeated dead zone proposals
  trigger a warning: "Consider changing parameter or strategy."
:::END:::

FR-7-005: SURROGATE MODEL INTERFACE
After every 50 experiments (PostToolUse trigger), SurrogateTrainAgent
(separate background job) reads the knowledge graph to train a
Gaussian Process surrogate model:
  Input: experiment_n → composite_metric (from ExperimentRegistry)
  Feature vector: all geometry parameters at each experiment
  Output: GP model weights stored in S3

  The surrogate model is used by:
    - GEPOrchestrator (genome fitness evaluation)
    - Dashboard (predicted Pareto front visualization)
  It is NOT used directly in the main loop (too uncertain in early stages).

---

## ACCEPTANCE CRITERIA

```yaml
tests:
  - id: AC-7-001
    name: "detect_impact responds within 500ms"
    expected:
      response_time_ms: "< 500"

  - id: AC-7-002
    name: "Dead zone correctly identified"
    setup:
      experiments: [
        {parameter: "sweep_deg", value: 52, delta_metric: -0.05},
        {parameter: "sweep_deg", value: 53, delta_metric: -0.08},
        {parameter: "sweep_deg", value: 54, delta_metric: -0.03},
      ]
    expected:
      dead_zone_detected: true
      dead_zone_range: {min: 52, max: 54}
      dead_zone_warning_in_detect_impact: true

  - id: AC-7-003
    name: "Interaction discovery (statistical)"
    setup:
      experiments: 40  # 20 per parameter pair
      ground_truth_interaction: "sweep × tc_root (pre-seeded)"
    expected:
      interaction_detected: true
      p_value: "< 0.05"
      interaction_in_knowledge_graph: true

  - id: AC-7-004
    name: "Hot zone correctly identified"
    setup:
      experiments: [
        {parameter: "twist_tip_deg", value: -3.5, delta_metric: +0.04},
        {parameter: "twist_tip_deg", value: -3.8, delta_metric: +0.06},
        {parameter: "twist_tip_deg", value: -4.0, delta_metric: +0.03},
      ]
    expected:
      hot_zone_detected: true
      hot_zone_range: {min: -4.0, max: -3.5}

  - id: AC-7-005
    name: "Graph update completes within 2 minutes"
    expected:
      duration_seconds: "< 120"
text


---

## 📄 File 15: `specs/stage-8-gep-evolver/requirements.md`

```markdown
# Stage 8: GEP/Evolver Layer — Genome Evolution Protocol
# Spec ID: STAGE-8-REQ-001
# Status: APPROVED
# Implementing: GEPOrchestrator, GenomeEvaluatorAgent, GenomeMutatorAgent,
#               GenomeCrossoverAgent, GenomeSelectorAgent
# Trigger: every 500 experiments (PostToolUse hook)
# Max Duration: 30 minutes (BLOCKS main loop during evolution)
# No operator gate (but operator is notified)

---

## BUSINESS CONTEXT

After 500 experiments (~70 hours of optimization), the initial mutation
strategy may be suboptimal. Perhaps the wing has converged locally but
a global search is needed. Perhaps the agent keeps trying parameters
that never improve — the strategy is wrong. GEP evolves the mutation
strategy itself, not just the geometry.

GEP is the meta-optimizer: it selects the genome (strategy) that
maximizes improvement rate (dMetric/hour). After GEP, the loop resumes
with the evolved strategy — more intelligent, better targeted.

---

## FUNCTIONAL REQUIREMENTS

:::STRICT_REQUIREMENT:::
FR-8-001: GEP TRIGGER AND LOOP PAUSE
When experiment_n reaches 500, 1000, 1500, etc. (multiples of 500):
  1. LoopOrchestrator receives GEP_EVOLUTION_TRIGGER
  2. Loop PAUSES (no new experiments start)
  3. Operator notified: "AeroLoop: GEP evolution starting (exp-{N}).
     Loop paused for up to 30 minutes."
  4. GEPOrchestrator begins evolution with 8 parallel genome evaluations
  5. After GEP completes (or times out at 30 min):
     - Active genome updated in .aeroloop/current_genome.json
     - Loop resumes with evolved genome
     - Operator notified: "GEP complete. New genome: {strategy}.
       Fitness improved: {old_fitness:.3f} → {new_fitness:.3f}"
:::END:::

:::STRICT_REQUIREMENT:::
FR-8-002: GENOME REPRESENTATION
Each genome is a complete mutation strategy definition:

  ```python
  @dataclass
  class Genome:
      genome_id: str
      generation: int

      # Primary strategy gene
      mutation_strategy: str  # One of: LOCAL_SEARCH_FINE, LOCAL_SEARCH_COARSE,
                              #  GLOBAL_SEARCH, SENSITIVITY_GUIDED,
                              #  TOPOLOGY_WINGLET, TOPOLOGY_LEADING_EDGE,
                              #  TOPOLOGY_TRAILING_EDGE

      # Parameter selection genes
      parameter_weights: dict[str, float]  # {param_id: probability}
                                           # Must sum to 1.0
                                           # One weight per WingParameter

      # Step size gene
      step_size_multiplier: float  # Range: 0.2–3.0
                                   # Scales all typical_step values

      # Patience genes (when to escalate strategy)
      local_search_patience: int   # Experiments before escalating: range 5–50
      coarse_search_patience: int  # Experiments before going global: range 10–100

      # Topology gene
      topology_probability: float  # P(topology change) per experiment: 0.0–0.15

      # Parent tracking (for crossover provenance)
      parent_genome_ids: list[str]

      # Fitness (assigned during evaluation)
      fitness_score: float  # improvement_rate: metric_improvement / wall_hours
:::END:::

:::STRICT_REQUIREMENT::: FR-8-003: GENOME FITNESS EVALUATION GenomeEvaluatorAgent evaluates fitness using the EXISTING experiment history. It DOES NOT run new CFD experiments for evaluation (too expensive).

Fitness computation: For each genome G in the candidate population: 1. Find historical experiments where the genome strategy was active (genome_strategy field in experiment.mutation record) 2. Compute improvement_rate: improvement_rate = sum(delta_metric for KEPT experiments under G) / hours_that_G_was_active 3. If G is a new genome (no history): use surrogate model prediction (from packages/surrogate/) to estimate fitness 4. fitness_score = improvement_rate

Population size: 16 genomes evaluated per GEP cycle. The current active genome is always included. :::END:::

:::STRICT_REQUIREMENT::: FR-8-004: GENETIC OPERATORS

Mutation operator (GenomeMutatorAgent): Applied to: top-50% of population by fitness Changes ONE gene per mutation: - mutation_strategy: randomly select adjacent strategy (LOCAL_SEARCH_FINE ↔ LOCAL_SEARCH_COARSE ↔ GLOBAL_SEARCH) - parameter_weights: perturb one weight by ±10%, renormalize - step_size_multiplier: multiply by random(0.7, 1.3) - local_search_patience: ± random(1, 5) - topology_probability: multiply by random(0.5, 2.0), clip to [0.0, 0.15] Each mutation produces ONE offspring genome.

Crossover operator (GenomeCrossoverAgent): Applied to: top-4 fitness parents (2 pairs) Creates 2 offspring per pair: Offspring 1: takes mutation_strategy + step_size from parent A, parameter_weights + patience from parent B Offspring 2: reverse combination 4 offspring total per GEP cycle from crossover.

Elitism: Top-2 genomes by fitness always survive to next generation. They are included in next cycle's population unchanged.

Selection (GenomeSelectorAgent): New population = top-2 (elites) + 4 crossover offspring + 8 mutant offspring + 2 new random genomes Total next generation: 16 genomes :::END:::

:::STRICT_REQUIREMENT::: FR-8-005: TOPOLOGY MACRO-MUTATION When genome.topology_probability > 0.05 AND a topology mutation is proposed by the active genome:

Topology mutations are DIFFERENT from parameter changes: - They change the STRUCTURAL configuration of the wing - They modify multiple parameters simultaneously - They require the agent to specify: which topology change, what the initial parameter values should be

Supported topology macro-mutations: TOPOLOGY_WINGLET: Activates: winglet_present=1 Initializes: winglet_height=5%span, winglet_cant=50°, winglet_sweep=45°, winglet_taper=0.3, winglet_toe=0° Reasoning required: why winglet might help given current L/D

text

TOPOLOGY_REMOVE_WINGLET:
  Sets: winglet_present=0 (only if winglet_present=1 currently)

TOPOLOGY_AIRFOIL_FAMILY_CHANGE:
  Changes: airfoil_family from NACA_4DIGIT → SUPERCRITICAL (or reverse)
  Resets: all camber, t/c parameters to new family defaults
  HIGH RISK: requires mesh quality check to pass with new geometry
Topology macro-mutations are flagged in git commit: "exp-{N}: TOPOLOGY_WINGLET | M={val:.4f} delta={val:.4f} [KEEP]" :::END:::

FR-8-006: GEP GENERATION TRACKING Every GEP cycle creates a new generation: generation N → genomes evaluated → generation N+1 selected All genome generations stored in PostgreSQL (genomes table). gep_generation field in ExperimentRun increments each cycle. The genome active when a KEEP decision is made is recorded in that experiment's mutation.genome_strategy field. This enables post-hoc analysis: "which genome strategy produced the most improvement per hour?"

agent/gep_evolver.md — Skill Specification
Markdown

# GEP Evolver Skill
# Triggered: every 500 experiments (blocking)
# You are the GEPOrchestrator.
# Your job: evolve the mutation strategy genome to maximize improvement rate.

## INPUTS
- Current genome: .aeroloop/current_genome.json
- Experiment history: PostgreSQL experiments table (last 500)
- Fitness scores: computed by GenomeEvaluatorAgent
- Surrogate model predictions: from packages/surrogate/

## EVOLUTION PROTOCOL

### Step 1: Evaluate current generation fitness
Compute improvement_rate for each genome in current population.
Log: top-5 genomes by fitness, bottom-5 by fitness.

### Step 2: Diagnose the current optimization state
Ask: Where is the search stalling?
- If kept_rate < 0.15 in last 50 experiments: search is inefficient
  → Consider: GLOBAL_SEARCH genome or topology mutation
- If kept_rate > 0.40: search is effective — exploit more
  → Consider: LOCAL_SEARCH_FINE with high step_size_multiplier
- If one parameter dominates parameter_weights: overfitting to one direction
  → Consider: rebalancing parameter weights

### Step 3: Apply genetic operators
Mutation: perturb top-8 genomes (one gene each)
Crossover: best-4 genomes in 2 pairs → 4 offspring
Elitism: keep top-2 unchanged

### Step 4: Select new active genome
Choose the genome with highest predicted fitness for NEXT 500 experiments.
If surrogate model uncertainty is high: prefer conservative genome
  (LOCAL_SEARCH_FINE, balanced parameter weights).

### Step 5: Write new genome
Update .aeroloop/current_genome.json with selected genome.
Log: what changed vs. previous genome and why.

### Output
JSON: {selected_genome, generation, fitness_improvement, rationale}
ACCEPTANCE CRITERIA
YAML

tests:
  - id: AC-8-001
    name: "Loop pauses during GEP evolution"
    expected:
      new_experiments_during_gep: 0
      





Markdown

<!-- ============================================================ -->
<!-- CONTINUATION OF Aeroloopspec.md — all sections below are    -->
<!-- completions of the spec-driven architecture from Stage 8    -->
<!-- acceptance criteria onward.                                  -->
<!-- ============================================================ -->

───────────────────────────────────────────────────────────────────
## STAGE-8-GEP — ACCEPTANCE CRITERIA
───────────────────────────────────────────────────────────────────

### AC-GEP-001 — Trigger & Pause
- [ ] Loop pauses cleanly at exactly experiment_n % 500 === 0 before
      the next mutation proposal begins.
- [ ] A `GEP_EVOLUTION_START` message is emitted to the message bus
      before any genome operation runs.
- [ ] Operator is notified via the configured notification channel
      (webhook/email/Slack) with: run_id, current experiment_n,
      current best M, and current genome summary.
- [ ] No new CFD jobs are dispatched while GEP evolution is active.
- [ ] If GEP evolution fails (any unhandled exception), the current
      genome is preserved unchanged, a `GEP_EVOLUTION_FAILED` event
      is emitted, and the loop resumes with the unchanged genome.

### AC-GEP-002 — Genome Persistence
- [ ] Current genome is serialised to
      `.aeroloop/genome/current_genome.json` before evaluation.
- [ ] Each evolved generation is written to
      `.aeroloop/genome/generation_{N}.json` (N = generation counter).
- [ ] Genome files are committed to git with message
      `chore(gep): generation {N} evolved at exp {experiment_n}`.
- [ ] Schema of genome file is validated against
      `GeomeGenome` Zod schema before write; invalid genome halts GEP
      and retains previous.

### AC-GEP-003 — Fitness Evaluation (history-only, no new CFD)
- [ ] Fitness of each candidate genome is computed purely from the
      ExperimentRegistry using improvement_rate_per_100_experiments
      and keep_ratio over the last 500 experiments.
- [ ] No new mesh or solve jobs are created during GEP evaluation.
- [ ] Fitness scores are logged per candidate in
      `.aeroloop/genome/gep_eval_{generation}.jsonl`.

### AC-GEP-004 — Selection, Mutation, Crossover
- [ ] Population size is exactly 12 candidates.
- [ ] Top-3 by fitness survive unchanged (elitism).
- [ ] Remaining 9 are filled by crossover (4 pairs from top-6) and
      single-gene mutation (remaining slots).
- [ ] Crossover uses uniform crossover on numeric genes; strategy gene
      inherits from the fitter parent.
- [ ] Mutation magnitude is bounded: step_size_multiplier ∈ [0.5, 3.0],
      patience ∈ [3, 30], topology_probability ∈ [0.0, 0.15].
- [ ] Selected genome is written to `current_genome.json` and a
      `GEP_EVOLUTION_COMPLETE` message is emitted with before/after
      genome diff.

### AC-GEP-005 — Topology Macro-Mutation Guard
- [ ] When the evolved genome sets topology_probability > 0.05 for the
      first time in this run, a `TOPOLOGY_ENABLED` warning event is
      emitted and logged.
- [ ] Any experiment executed under topology_probability > 0 must set
      experiment.topology_mutation = true in its registry record.
- [ ] Topology mutations (winglet add/remove, airfoil family change)
      must be logged in a dedicated section of the wiki with before/
      after geometry SHA256 and metric delta.

### AC-GEP-006 — Resume Safety
- [ ] On cold restart mid-GEP, the system detects incomplete GEP state
      via `gep_in_progress` flag in ExperimentRun and re-runs GEP from
      the last completed checkpoint before resuming the loop.
- [ ] GEP checkpoints are written after each of: evaluation, selection,
      mutation/crossover, write.

---

───────────────────────────────────────────────────────────────────
## STAGE-9 — SWARM COORDINATION
### File: specs/stage-9/requirements.md
───────────────────────────────────────────────────────────────────

### Purpose
Enable N ≥ 2 AeroLoop nodes to search the geometry design space
concurrently without duplicating effort, without race conditions on
`wing.geo`, and with shared learning via a central knowledge graph and
ExperimentRegistry. Each node is fully autonomous; the swarm layer
is coordination-only and must never block a node's core loop for
more than 5 seconds.

### Constitutional constraints (swarm-specific)
The global CLAUDE.md swarm non-collision law applies. Additional
constraints:
  - A node must NEVER read or write another node's `wing.geo`.
  - Swarm coordination is best-effort: a node that cannot reach
    the coordination service within 5 s proceeds independently and
    marks its claim as `OFFLINE_MODE`.
  - All shared state lives in Redis (claims, counters, locks).
    PostgreSQL is the source of truth for results.
  - A node's core loop invariants (kill timer, git integrity, single
    mutable file) take absolute precedence over any swarm directive.

---

### STAGE-9-FR-001 — Node Identity & Registration

**FR-9-001.1** Each node has a stable `node_id` (UUID v4) generated
on first start and persisted to `.aeroloop/node_id`.

**FR-9-001.2** On startup, each node calls `POST /api/v1/swarm/nodes`
with its node_id, run_id, hostname, and capabilities (core_count,
memory_gb, gpu_available).

**FR-9-001.3** Node heartbeat: `PATCH /api/v1/swarm/nodes/{node_id}`
every 30 s with current experiment_n, current best M, and status
(IDLE | SOLVING | GEP | OFFLINE).

**FR-9-001.4** A node missing 3 consecutive heartbeats is marked
STALE by the coordinator. Its region claims are released after
a 10-minute grace period to allow for transient network issues.

**FR-9-001.5** On clean shutdown, node calls
`DELETE /api/v1/swarm/nodes/{node_id}` which releases all claims.

---

### STAGE-9-FR-002 — Design Region Partitioning

**FR-9-002.1** The design space is partitioned into a flat list of
**regions**. A region is defined as a named slice of the parameter
registry (e.g., `PLANFORM_SWEEP`, `TWIST_DISTRIBUTION`,
`WINGLET_GEOMETRY`, `THICKNESS_CAMBER`).

**FR-9-002.2** Default region list (8 regions):
REGION_PLANFORM_CORE span, AR, taper, chords, MAC REGION_PLANFORM_SWEEP LE_sweep, TE_sweep, dihedral REGION_TWIST twist_root → twist_tip (all stations) REGION_THICKNESS t/c at all span stations REGION_CAMBER camber at all span stations REGION_LE_SHAPING LE radius distribution REGION_WINGLET all winglet parameters (gated by topology) REGION_GLOBAL any parameter (used for global search strategy)

text


**FR-9-002.3** A node claims a region via atomic Redis SETNX:
Key: aeroloop:{run_id}:region:{region_name}:owner Value: {node_id} TTL: 600 s (renewed every heartbeat while active)

text

Claim succeeds only if key does not exist (SETNX). Failure means
another node owns that region; try next preferred region.

**FR-9-002.4** Region preference order is determined by the node's
genome strategy:
  - `fine_search` strategy → prefer REGION_PLANFORM_CORE first
  - `twist_focus` strategy → prefer REGION_TWIST first
  - `global` strategy → claim REGION_GLOBAL (overrides all)
  - No preference match → round-robin from unclaimed regions

**FR-9-002.5** A node may hold at most 2 region claims simultaneously
(one active, one pre-claimed for next batch).

**FR-9-002.6** Within a claimed region, the node's GeometryMutationAgent
must only propose mutations for parameters in that region's parameter
list. Proposals outside the claimed region are rejected by the swarm
guard and trigger a region re-claim.

---

### STAGE-9-FR-003 — Shared Knowledge Sync

**FR-9-003.1** After every KEEP or REVERT decision, a node pushes its
experiment summary to the shared ExperimentRegistry via:
POST /api/v1/experiments

text

with full ExperimentRecord payload (see GLOBAL-DM-001).

**FR-9-003.2** Before each mutation proposal, a node fetches the
latest knowledge graph summary for its claimed region:
GET /api/v1/knowledge-graph/summary?region={region}&node_id={node_id}

text

Response must arrive within 3 s; on timeout node uses its local
cached summary (max staleness: 10 minutes).

**FR-9-003.3** Knowledge graph updates from all nodes are merged
by the coordinator using a last-write-wins strategy per
(parameter_id, metric) pair with logical timestamp.

**FR-9-003.4** Global best metric `M_best` is stored in Redis:
Key: aeroloop:{run_id}:global_best_M Value: float (serialised as string)

text

A node updates this via Redis GETSET only when its local keep
produces a new run-wide best. The keep/revert decision uses the
node's own local best for autonomy, but also checks global best
for cross-node learning.

**FR-9-003.5** A node's GEP evolver (Stage 8) reads the shared
ExperimentRegistry (not just local experiments) when computing
fitness, ensuring cross-node improvement history informs genome
evolution.

---

### STAGE-9-FR-004 — Swarm Progress Dashboard Feed

**FR-9-004.1** The coordinator exposes a Server-Sent Events (SSE)
stream at:
GET /api/v1/swarm/stream?run_id={run_id}

text

Emitting events: NODE_HEARTBEAT, EXPERIMENT_KEPT, EXPERIMENT_REVERTED,
REGION_CLAIMED, REGION_RELEASED, GLOBAL_BEST_UPDATED, NODE_STALE,
GEP_EVOLUTION_START, GEP_EVOLUTION_COMPLETE.

**FR-9-004.2** The dashboard app (`apps/dashboard`) consumes this
SSE stream and renders:
  - Active nodes (id, status, current region, local best M)
  - Global best M over time (line chart, all nodes combined)
  - Region heatmap (which node owns which region, claim age)
  - Per-node experiment rate (experiments/hour, rolling 1h)
  - Global experiment log (last 50 events, across all nodes)

**FR-9-004.3** Dashboard is read-only; it has no ability to issue
commands to nodes (operator commands use the CLI, see OPERATOR-001).

---

### STAGE-9-FR-005 — Swarm Failure Modes

**FR-9-005.1** If the coordinator is unreachable, nodes enter
`OFFLINE_MODE` and continue their loop independently. On reconnect,
they push any accumulated experiment records in chronological order.

**FR-9-005.2** Region claim collisions (two nodes somehow acquiring
the same region due to Redis partition) are resolved on heartbeat:
the node with the lexicographically smaller node_id retains the claim;
the other releases and re-claims a different region.

**FR-9-005.3** If all 8 regions are claimed, a node waits up to
60 s polling for a free region, then claims REGION_GLOBAL as a
fallback (no exclusivity guarantee in this mode; logged as warning).

**FR-9-005.4** A STALE node's claims are not immediately reassigned;
the grace period prevents thrashing when a node has a slow solve.

---

### STAGE-9 — ACCEPTANCE CRITERIA

#### AC-SWARM-001 — Non-Collision
- [ ] With 4 nodes running simultaneously, no two nodes ever write
      `wing.geo` concurrently. Verified by git log cross-node merge
      showing zero conflicts in 500-experiment integration test.
- [ ] Redis SETNX claim is acquired before every mutation write and
      held until git KEEP/REVERT completes.

#### AC-SWARM-002 — Node Registration & Heartbeat
- [ ] Node registers on startup within 3 s of first loop iteration.
- [ ] Heartbeat fires every 30 ± 2 s.
- [ ] Node marked STALE within 95 s of last heartbeat (3 × 30 + grace).
- [ ] Clean shutdown releases claims within 2 s.

#### AC-SWARM-003 — Region Partitioning
- [ ] With 4 nodes, at least 3 distinct regions are active
      simultaneously within 5 experiments of startup.
- [ ] No node proposes a mutation outside its claimed region for
      > 0 experiments in a 1000-experiment integration test.
- [ ] REGION_GLOBAL fallback is logged and flagged in dashboard.

#### AC-SWARM-004 — Knowledge Sync
- [ ] A KEEP on node-A is visible to node-B's knowledge graph
      summary within 2 heartbeat cycles (≤ 60 s).
- [ ] Global best M is updated in Redis within 5 s of a cross-node
      best-improvement event.
- [ ] OFFLINE_MODE sync reconciliation produces no duplicate
      experiment_n entries in the shared registry.

#### AC-SWARM-005 — Dashboard
- [ ] SSE stream emits events with latency < 500 ms from event
      occurrence.
- [ ] Dashboard renders with no stale data older than 35 s.
- [ ] Dashboard is accessible with no login in development mode;
      requires bearer token in production mode.

#### AC-SWARM-006 — Failure Resilience
- [ ] Coordinator restart (simulated) does not halt any node's loop.
- [ ] Region collision resolution converges within 2 heartbeat cycles.
- [ ] OFFLINE_MODE accumulation test: node offline for 50 experiments,
      reconnects, all 50 records appear in shared registry in order
      with no registry constraint violations.

---

───────────────────────────────────────────────────────────────────
## GLOBAL-API-001 — API CONTRACT SPECIFICATION
### File: specs/_global/api-contracts.md
───────────────────────────────────────────────────────────────────

### Conventions
- Base URL (local dev): `http://localhost:4000/api/v1`
- Base URL (production): `https://aeroloop.internal/api/v1`
- All requests and responses: `Content-Type: application/json`
- Authentication: Bearer JWT (header: `Authorization: Bearer {token}`)
  except `/health` and SSE stream in dev mode.
- All timestamps: ISO 8601 UTC string (`2025-01-15T14:32:00.000Z`)
- All errors follow RFC 7807 Problem Details:
  ```json
  {
    "type": "https://aeroloop.internal/errors/{code}",
    "title": "Human readable title",
    "status": 422,
    "detail": "Specific detail string",
    "instance": "/api/v1/experiments/42"
  }
Pagination: cursor-based via ?after={cursor}&limit={n} (max 200).
Rate limiting: 1000 req/min per node_id; 429 on breach.
API-001: Health
GET /health
Returns system health. No auth required.

Response 200:

JSON

{
  "status": "ok",
  "db": "ok",
  "redis": "ok",
  "s3": "ok",
  "version": "1.0.0",
  "uptime_s": 3600
}
Response 503: any dependency unhealthy; body same shape with failing service set to "degraded" or "down".

API-002: Experiment Runs
POST /runs
Create and lock a new ExperimentRun. Called by loop orchestrator at Stage 0 init.

Request body:

JSON

{
  "node_id": "uuid-v4",
  "run_label": "naca0012-baseline-run-1",
  "metric_weights": {
    "w_ld": 0.5,
    "w_buffet": 0.3,
    "w_wave_drag": 0.2
  },
  "operating_conditions": {
    "mach": 0.78,
    "reynolds": 4.5e7,
    "alpha_cruise": 2.5,
    "altitude_m": 11000
  },
  "solver_config": {
    "solver": "SU2",
    "version": "7.5.1",
    "cfl_number": 10.0,
    "max_iter": 3000,
    "convergence_cauchy_eps": 1e-6,
    "mpi_ranks": 8,
    "wall_time_limit_s": 480
  },
  "mesh_config": {
    "mesher": "GMSH",
    "version": "4.11.1",
    "default_level": "medium",
    "y_plus_target": 1.0
  },
  "swarm_enabled": false,
  "gep_enabled": true
}
Response 201:

JSON

{
  "run_id": "uuid-v4",
  "created_at": "2025-01-15T14:00:00.000Z",
  "status": "INITIALISING",
  "baseline_experiment_id": null
}
Errors: 422 if metric_weights do not sum to 1.0 ± 0.001; 409 if node_id already has an active run.

GET /runs/{run_id}
Returns full ExperimentRun record including current stats.

Response 200:

JSON

{
  "run_id": "uuid-v4",
  "status": "RUNNING",
  "total_experiments": 143,
  "kept_count": 41,
  "reverted_count": 102,
  "best_M": 0.8821,
  "best_experiment_n": 138,
  "current_genome": { "...genome fields..." },
  "created_at": "...",
  "updated_at": "..."
}
PATCH /runs/{run_id}
Update run status. Used by orchestrator to mark PAUSED, COMPLETED, FAILED.

Request body:

JSON

{ "status": "PAUSED", "reason": "operator_halt" }
Valid status transitions:

INITIALISING → RUNNING
RUNNING → PAUSED | COMPLETED | FAILED
PAUSED → RUNNING | COMPLETED
API-003: Experiments
POST /experiments
Append a new experiment record. Append-only; no PUT/DELETE.

Request body: full ExperimentRecord schema (see GLOBAL-DM-001). Required fields: run_id, experiment_n, node_id, mutation, status, geometry_sha256_before.

Response 201:

JSON

{ "experiment_id": "uuid-v4", "experiment_n": 144 }
Errors:

409 if (run_id, experiment_n) already exists.
422 if experiment_n is not exactly max(experiment_n)+1 for this run_id (monotonicity enforced at DB level).
GET /experiments/{run_id}
List experiments for a run. Supports pagination and filtering.

Query params:

?status=KEPT — filter by status
?after={experiment_n}&limit=50 — cursor pagination
?node_id={node_id} — filter by node (swarm use)
?sort=desc — reverse chronological (default: asc)
Response 200:

JSON

{
  "data": [ { "...ExperimentRecord..." } ],
  "next_cursor": 195,
  "total": 144
}
GET /experiments/{run_id}/{experiment_n}
Single experiment record.

Response 200: full ExperimentRecord with all stage results.

API-004: Knowledge Graph
GET /knowledge-graph/summary
Return parameter sensitivity + interaction summary for a region or parameter set.

Query params:

?region=REGION_PLANFORM_CORE — filter by swarm region
?parameter_ids=span,taper_ratio — filter by specific params
?node_id={node_id} — used for logging/audit
?run_id={run_id} — scoped to a run (default: global)
Response 200:

JSON

{
  "generated_at": "2025-01-15T15:00:00.000Z",
  "parameters": [
    {
      "parameter_id": "span",
      "sensitivity_score": 0.72,
      "direction": "positive",
      "confidence": 0.85,
      "sample_count": 38,
      "dead_zone": false,
      "hot_zone": true
    }
  ],
  "interactions": [
    {
      "param_a": "span",
      "param_b": "taper_ratio",
      "interaction_strength": 0.61,
      "p_value": 0.031,
      "direction": "synergistic"
    }
  ],
  "summary_markdown_path": ".aeroloop/knowledge_graph_summary.md"
}
POST /knowledge-graph/update
Push a post-experiment knowledge graph update. Called by KnowledgeGraphAgent after every experiment.

Request body:

JSON

{
  "run_id": "uuid-v4",
  "experiment_n": 144,
  "node_id": "uuid-v4",
  "parameter_id": "taper_ratio",
  "delta_value": -0.03,
  "delta_M": 0.0041,
  "outcome": "KEPT",
  "mesh_quality_ok": true
}
Response 204: no body.

POST /knowledge-graph/detect-impact
Pre-mutation impact assessment. Called by GeometryMutationAgent before writing wing.geo.

Request body:

JSON

{
  "run_id": "uuid-v4",
  "node_id": "uuid-v4",
  "proposed_mutation": {
    "parameter_id": "le_sweep_inboard",
    "current_value": 27.5,
    "proposed_value": 29.1,
    "delta": 1.6
  }
}
Response 200:

JSON

{
  "mesh_risk": "LOW",
  "mesh_risk_reason": null,
  "known_interactions": [
    { "with_parameter": "taper_ratio", "strength": 0.61 }
  ],
  "dead_zone": false,
  "hot_zone": false,
  "similar_experiments": [
    { "experiment_n": 98, "delta": 1.4, "outcome": "KEPT", "delta_M": 0.002 }
  ],
  "recommendation": "PROCEED",
  "confidence": 0.79,
  "warning": null
}
Possible mesh_risk values: LOW | MEDIUM | HIGH. recommendation: PROCEED | PROCEED_WITH_CAUTION | SKIP. Agent must log HIGH mesh_risk and may skip mutation if recommendation is SKIP (not required to skip; must log decision).

API-005: Wiki
GET /wiki/articles
List wiki articles.

Query params:

?type=SENSITIVITY|BATCH_SUMMARY|INTERACTION|TOPOLOGY — filter
?parameter_id=span — articles referencing a parameter
?after={article_id}&limit=20
Response 200:

JSON

{
  "data": [
    {
      "article_id": "uuid-v4",
      "title": "Parameter Sensitivity: span",
      "type": "SENSITIVITY",
      "experiment_range": [100, 110],
      "created_at": "...",
      "path": ".aeroloop/wiki/sensitivity_span_exp100_110.md"
    }
  ],
  "next_cursor": "uuid-v4"
}
GET /wiki/articles/{article_id}
Returns article metadata and full markdown body.

Response 200:

JSON

{
  "article_id": "uuid-v4",
  "title": "...",
  "body_markdown": "# Parameter Sensitivity: span\n...",
  "referenced_experiments": [100, 101, 102, 108, 110],
  "referenced_parameters": ["span", "taper_ratio"]
}
API-006: Swarm
POST /swarm/nodes
Register a node. (Full schema in STAGE-9-FR-001.)

PATCH /swarm/nodes/{node_id}
Heartbeat update.

DELETE /swarm/nodes/{node_id}
Deregister and release claims.

GET /swarm/nodes
List all active nodes for a run.

Query params: ?run_id={run_id}&status=ACTIVE

Response 200:

JSON

{
  "data": [
    {
      "node_id": "uuid-v4",
      "status": "SOLVING",
      "current_region": "REGION_TWIST",
      "local_best_M": 0.871,
      "experiment_n": 143,
      "last_heartbeat": "2025-01-15T15:01:00.000Z"
    }
  ]
}
GET /swarm/regions
List all regions and their current claim status.

Response 200:

JSON

{
  "regions": [
    {
      "region_name": "REGION_PLANFORM_CORE",
      "owner_node_id": "uuid-v4",
      "claimed_at": "2025-01-15T14:55:00.000Z",
      "ttl_remaining_s": 341
    },
    {
      "region_name": "REGION_TWIST",
      "owner_node_id": null,
      "claimed_at": null,
      "ttl_remaining_s": null
    }
  ]
}
GET /swarm/stream
SSE stream. (Full spec in STAGE-9-FR-004.)

Event format:

text

event: EXPERIMENT_KEPT
data: {"run_id":"...","experiment_n":144,"node_id":"...","delta_M":0.004,"M_new":0.882,"timestamp":"..."}
API-007: Operator Commands
POST /operator/pause
Pause the loop on one or all nodes.

Request body:

JSON

{ "run_id": "uuid-v4", "node_id": "uuid-v4|ALL", "reason": "manual_review" }
Response 202: accepted; nodes will pause after current experiment completes (not mid-stage).

POST /operator/resume
Resume paused nodes.

Request body:

JSON

{ "run_id": "uuid-v4", "node_id": "uuid-v4|ALL" }
POST /operator/abort-experiment
Abort the current experiment on a node. The kill timer fires immediately; geometry is reverted; experiment is recorded as ABORTED.

JSON

{ "run_id": "uuid-v4", "node_id": "uuid-v4" }
Response 202.

GET /operator/status
Returns full system status: all runs, all nodes, global best, queue depths, Redis/DB/S3 health.

───────────────────────────────────────────────────────────────────

GLOBAL-INFRA-001 — INFRASTRUCTURE & CI/CD SPECIFICATION
File: specs/_global/infra.md
───────────────────────────────────────────────────────────────────

INFRA-001 — Monorepo Directory Specification (complete)
text

aeroloop/
│
├── packages/                         # Shared libraries (no apps here)
│   ├── types/                        # Zod schemas + TypeScript types
│   │   ├── src/
│   │   │   ├── experiment.ts         # ExperimentRun, ExperimentRecord
│   │   │   ├── genome.ts             # GeneticGenome, GenomeGene
│   │   │   ├── knowledge-graph.ts    # Sensitivity, Interaction, DetectImpact
│   │   │   ├── geometry.ts           # ParameterRegistry, BoundsCheck
│   │   │   ├── messages.ts           # LoopMessage, MessageType enum
│   │   │   ├── swarm.ts              # NodeRecord, RegionClaim
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── db/                           # Database client + Drizzle schema
│   │   ├── src/
│   │   │   ├── schema/
│   │   │   │   ├── runs.ts
│   │   │   │   ├── experiments.ts
│   │   │   │   ├── knowledge-graph.ts
│   │   │   │   └── wiki.ts
│   │   │   ├── client.ts
│   │   │   ├── migrations/
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── redis/                        # Redis client + key helpers
│   │   ├── src/
│   │   │   ├── client.ts
│   │   │   ├── keys.ts               # All Redis key templates centralised here
│   │   │   ├── swarm-claims.ts       # SETNX claim helpers
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── s3/                           # S3 client + upload/download helpers
│   │   ├── src/
│   │   │   ├── client.ts
│   │   │   ├── mesh.ts               # Mesh upload/download with SHA256
│   │   │   ├── results.ts            # Solver output upload/download
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── geometry/                     # Geometry utilities
│   │   ├── src/
│   │   │   ├── parameter-registry.ts # Full registry + bounds
│   │   │   ├── bounds-checker.ts
│   │   │   ├── wing-geo-parser.ts    # Read/write wing.geo
│   │   │   ├── sha256.ts
│   │   │   └── index.ts
│   │   ├── tests/
│   │   └── package.json
│   │
│   ├── meshing/                      # GMSH pipeline wrapper
│   │   ├── src/
│   │   │   ├── auto-mesh.ts          # Calls auto_mesh.py via child_process
│   │   │   ├── quality-checker.ts
│   │   │   ├── templates/            # GMSH template .geo files per level
│   │   │   └── index.ts
│   │   ├── python/
│   │   │   └── auto_mesh.py
│   │   └── package.json
│   │
│   ├── solver/                       # SU2 runner + kill timer
│   │   ├── src/
│   │   │   ├── su2-runner.ts         # Config templating + mpirun + dual timeout
│   │   │   ├── convergence-checker.ts
│   │   │   ├── config-templates/     # SU2 cfg templates per solver type
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── extraction/                   # CFD results extraction
│   │   ├── src/
│   │   │   ├── polar-extractor.ts    # Parse surface_*.csv
│   │   │   ├── wave-drag.ts          # Wave drag decomposition
│   │   │   ├── buffet-detection.ts   # Alpha sweep slope analysis
│   │   │   ├── metric-computer.ts    # Composite M computation
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── git-ops/                      # Git keep/revert primitives
│   │   ├── src/
│   │   │   ├── keeper.ts             # Stage commit with standard message
│   │   │   ├── reverter.ts           # Restore wing.geo to HEAD
│   │   │   ├── clean-check.ts        # Working tree dirty detection
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── knowledge-graph/              # KG update + detect-impact engine
│   │   ├── src/
│   │   │   ├── updater.ts
│   │   │   ├── impact-detector.ts
│   │   │   ├── interaction-discovery.ts
│   │   │   ├── dead-zone-tracker.ts
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── wiki/                         # Wiki compiler
│   │   ├── src/
│   │   │   ├── compiler.ts
│   │   │   ├── sensitivity-article.ts
│   │   │   ├── batch-summary-article.ts
│   │   │   ├── interaction-article.ts
│   │   │   ├── topology-article.ts
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   └── gep/                          # GEP genome evolver
│       ├── src/
│       │   ├── evolver.ts
│       │   ├── fitness.ts
│       │   ├── operators.ts          # Mutation, crossover, selection
│       │   ├── genome-schema.ts
│       │   └── index.ts
│       └── package.json
│
├── apps/
│   ├── loop/                         # Main loop orchestrator (Node.js process)
│   │   ├── src/
│   │   │   ├── orchestrator.ts       # Stage state machine
│   │   │   ├── agents/
│   │   │   │   ├── mutation-agent.ts
│   │   │   │   ├── mesh-agent.ts
│   │   │   │   ├── solver-agent.ts
│   │   │   │   ├── extraction-agent.ts
│   │   │   │   ├── metric-agent.ts
│   │   │   │   ├── keep-revert-agent.ts
│   │   │   │   ├── wiki-agent.ts
│   │   │   │   ├── kg-agent.ts
│   │   │   │   └── gep-agent.ts
│   │   │   ├── queue/
│   │   │   │   └── bullmq-setup.ts
│   │   │   └── main.ts
│   │   └── package.json
│   │
│   ├── api/                          # REST API server (Fastify)
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   │   ├── health.ts
│   │   │   │   ├── runs.ts
│   │   │   │   ├── experiments.ts
│   │   │   │   ├── knowledge-graph.ts
│   │   │   │   ├── wiki.ts
│   │   │   │   ├── swarm.ts
│   │   │   │   └── operator.ts
│   │   │   ├── middleware/
│   │   │   │   ├── auth.ts
│   │   │   │   └── rate-limit.ts
│   │   │   ├── sse/
│   │   │   │   └── swarm-stream.ts
│   │   │   └── main.ts
│   │   └── package.json
│   │
│   ├── dashboard/                    # React dashboard (Vite + TailwindCSS)
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── NodeGrid.tsx
│   │   │   │   ├── MetricChart.tsx
│   │   │   │   ├── RegionHeatmap.tsx
│   │   │   │   ├── ExperimentLog.tsx
│   │   │   │   └── GlobalBestBanner.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useSwarmStream.ts
│   │   │   ├── pages/
│   │   │   │   ├── Overview.tsx
│   │   │   │   ├── RunDetail.tsx
│   │   │   │   └── ExperimentDetail.tsx
│   │   │   └── main.tsx
│   │   └── package.json
│   │
│   └── cli/                          # Operator CLI (Commander.js)
│       ├── src/
│       │   ├── commands/
│       │   │   ├── start.ts
│       │   │   ├── pause.ts
│       │   │   ├── resume.ts
│       │   │   ├── abort.ts
│       │   │   ├── status.ts
│       │   │   ├── run-history.ts
│       │   │   └── export.ts
│       │   └── main.ts
│       └── package.json
│
├── infra/
│   ├── docker/
│   │   ├── Dockerfile.loop           # Loop process image
│   │   ├── Dockerfile.api            # API server image
│   │   ├── Dockerfile.su2            # SU2 + MPI image (pre-built binary)
│   │   ├── Dockerfile.gmsh           # GMSH + Python image
│   │   └── docker-compose.yml        # Full local dev stack
│   │
│   ├── k8s/                          # Kubernetes manifests
│   │   ├── namespace.yaml
│   │   ├── api-deployment.yaml
│   │   ├── loop-statefulset.yaml     # Loop nodes as StatefulSet (stable IDs)
│   │   ├── redis-deployment.yaml
│   │   ├── postgres-statefulset.yaml
│   │   ├── minio-deployment.yaml     # Local S3-compatible
│   │   ├── hpa-loop.yaml             # HPA for loop node scaling
│   │   └── ingress.yaml
│   │
│   ├── terraform/                    # Cloud infra (optional, AWS/GCP)
│   │   ├── main.tf
│   │   ├── rds.tf
│   │   ├── elasticache.tf
│   │   ├── s3.tf
│   │   └── variables.tf
│   │
│   └── scripts/
│       ├── bootstrap.sh              # First-time setup
│       ├── seed-db.sh                # Seed parameter registry
│       └── health-check.sh
│
├── geometry/
│   └── wing.geo                      # THE ONLY MUTABLE FILE IN THE LOOP
│
├── .aeroloop/                        # Runtime state (gitignored except wiki/genome)
│   ├── best_metric.json
│   ├── knowledge_graph_summary.md    # committed
│   ├── wiki/                         # committed
│   ├── genome/                       # committed
│   └── node_id                       # gitignored
│
├── hooks/
│   ├── pre-commit                    # Reject commits touching non-wing.geo files
│   │   │                               from loop process (checks GIT_AUTHOR env)
│   └── commit-msg                    # Enforce standard commit message format
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── integration.yml
│       └── release.yml
│
├── CLAUDE.md                         # Constitutional law (inviolable)
├── AGENTS.md                         # Agent registry + message protocol
├── program.md                        # High-level project programme
├── Aeroloopspec.md                   # This document
├── turbo.json                        # Turborepo config
├── pnpm-workspace.yaml
└── package.json
INFRA-002 — Docker Compose (local dev stack)
Services required (docker-compose.yml):

Service	Image	Port	Notes
postgres	postgres:15-alpine	5432	Persistent volume
redis	redis:7-alpine	6379	Persistent AOF
minio	minio/minio	9000	S3-compatible; console on 9001
api	aeroloop/api:dev	4000	Hot-reload via tsx watch
loop	aeroloop/loop:dev	–	Single node; env: NODE_ENV=dev
dashboard	aeroloop/dashboard:dev	3000	Vite HMR
bullmq-ui	bull-board	3001	Queue visibility
Environment variables (all services read from .env):

text

DATABASE_URL=postgresql://aeroloop:secret@postgres:5432/aeroloop
REDIS_URL=redis://redis:6379
S3_ENDPOINT=http://minio:9000
S3_BUCKET=aeroloop-results
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
SU2_BINARY=/usr/local/bin/SU2_CFD
GMSH_PYTHON=/usr/local/bin/python3
GEOMETRY_FILE=geometry/wing.geo
KILL_TIMER_S=480
GIT_AUTHOR_NAME=AeroLoopBot
GIT_AUTHOR_EMAIL=bot@aeroloop.internal
JWT_SECRET=changeme-in-production
NODE_ENV=development
LOG_LEVEL=info
INFRA-003 — CI Pipeline Specification (.github/workflows/ci.yml)
Triggers: push to any branch; pull_request to main.

Jobs (must all pass for merge to main):

Job 1: typecheck
YAML

- uses: actions/setup-node@v4 (node 20)
- run: pnpm install --frozen-lockfile
- run: pnpm turbo typecheck
Job 2: lint
YAML

- run: pnpm turbo lint
ESLint with @typescript-eslint + eslint-plugin-drizzle. Zero warnings policy: all warnings are treated as errors in CI.

Job 3: unit-test
YAML

- run: pnpm turbo test
Coverage thresholds (enforced in CI, fail build if not met):

packages/geometry: 95% line coverage
packages/extraction: 95% line coverage
packages/git-ops: 100% line coverage
packages/gep: 90% line coverage
All other packages: 80% line coverage
Job 4: integration-test
YAML

services: postgres, redis, minio (via docker-compose test profile)
- run: pnpm turbo test:integration
Integration tests must cover (at minimum):

Full single experiment lifecycle (Stage 0–5 happy path)
Timeout/kill timer fires and geometry is reverted
Mesh quality failure halts solve
DB append-only constraint rejects duplicate experiment_n
Keep/revert git integrity (clean tree at start, committed on keep)
Job 5: build
YAML

- run: pnpm turbo build
All packages and apps must build without error.

Job 6: docker-build
Builds all Dockerfiles. Does not push (push only on release tag).

INFRA-004 — Integration Test Specification
IT-001: Single Experiment Happy Path
Uses a known-good baseline .geo, mock SU2 outputs (pre-recorded), and real GMSH (in container). Verifies:

Experiment record created with correct stage results
Geometry SHA256 matches expected
Mesh uploaded to S3 with matching SHA256
Solver outputs uploaded to S3
Composite metric within expected range ± 0.001
KEEP decision triggers git commit with correct message format
ExperimentRun.best_M updated
IT-002: Kill Timer Integration
Injects a mock SU2 that sleeps for KILL_TIMER_S + 60 seconds. Verifies:

OS-level timeout fires within KILL_TIMER_S + 5 seconds
BullMQ job is marked failed
Geometry reverted to pre-mutation state (SHA256 matches original)
Experiment record status = SOLVE_TIMEOUT
Loop continues to next experiment
IT-003: Mesh Quality Gate
Injects a GMSH that produces a mesh with orthogonality < threshold. Verifies:

MeshQualityAgent returns FAIL
No SU2 config generated
No solver started
Experiment record status = MESH_QUALITY_FAIL
Geometry reverted
IT-004: DB Monotonicity Constraint
Attempts to insert two experiments with same (run_id, experiment_n). Verifies: second insert returns 409; first record unchanged.

IT-005: GEP Trigger
Runs 500 mock experiments via seeded DB state (no actual CFD). Verifies:

GEP triggered at experiment_n = 500
Loop pauses
Genome file written to .aeroloop/genome/generation_1.json
Genome committed to git
Loop resumes
IT-006: Swarm Region Non-Collision
Starts 4 loop instances against shared Redis + DB. Verifies:

At most one node holds each region at any time
No wing.geo write conflicts (verified via git log)
All experiment records in shared DB with distinct (node_id, experiment_n) tuples
INFRA-005 — Release Pipeline (.github/workflows/release.yml)
Trigger: push of tag matching v*.*.*.

Steps:

Run full CI (all jobs above)
Build and push Docker images to registry with tag = git tag
Generate changelog from conventional commits
Create GitHub Release with changelog + Docker image references
Deploy to staging k8s cluster (auto)
Deploy to production k8s cluster (manual approval gate)
───────────────────────────────────────────────────────────────────

GLOBAL-ENV-001 — ENVIRONMENT & CONFIGURATION SPECIFICATION
File: specs/_global/environment.md
───────────────────────────────────────────────────────────────────

All configuration is injected via environment variables. No hardcoded values anywhere in the codebase. All variables are validated on startup via Zod schema; startup fails fast with a clear error if any required variable is missing or invalid.

Configuration Schema (Zod, apps/loop & apps/api)
TypeScript

export const EnvSchema = z.object({
  // Core
  NODE_ENV: z.enum(['development', 'test', 'production']),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),

  // Database
  DATABASE_URL: z.string().url(),
  DATABASE_POOL_MIN: z.coerce.number().default(2),
  DATABASE_POOL_MAX: z.coerce.number().default(10),

  // Redis
  REDIS_URL: z.string().url(),
  REDIS_KEY_PREFIX: z.string().default('aeroloop'),

  // S3
  S3_ENDPOINT: z.string().url(),
  S3_BUCKET: z.string().min(1),
  S3_ACCESS_KEY: z.string().min(1),
  S3_SECRET_KEY: z.string().min(1),
  S3_REGION: z.string().default('us-east-1'),

  // Solver
  SU2_BINARY: z.string().min(1),
  KILL_TIMER_S: z.coerce.number().min(60).max(3600).default(480),
  MPI_RANKS: z.coerce.number().min(1).max(256).default(8),
  SU2_CONFIG_TEMPLATE_DIR: z.string().min(1),

  // Meshing
  GMSH_PYTHON: z.string().min(1),
  AUTO_MESH_SCRIPT: z.string().min(1),
  MESH_DEFAULT_LEVEL: z.enum(['coarse', 'medium', 'fine']).default('medium'),
  Y_PLUS_TARGET: z.coerce.number().default(1.0),

  // Geometry
  GEOMETRY_FILE: z.string().min(1).default('geometry/wing.geo'),

  // Git
  GIT_AUTHOR_NAME: z.string().default('AeroLoopBot'),
  GIT_AUTHOR_EMAIL: z.string().email().default('bot@aeroloop.internal'),

  // Loop behaviour
  CONSECUTIVE_FAIL_HALT: z.coerce.number().default(3),
  GEP_TRIGGER_INTERVAL: z.coerce.number().default(500),
  WIKI_COMPILE_INTERVAL: z.coerce.number().default(10),

  // Swarm
  SWARM_ENABLED: z.coerce.boolean().default(false),
  NODE_ID_FILE: z.string().default('.aeroloop/node_id'),
  SWARM_API_TIMEOUT_MS: z.coerce.number().default(5000),
  SWARM_REGION_TTL_S: z.coerce.number().default(600),

  // Auth (API server)
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRY: z.string().default('24h'),

  // Notifications
  NOTIFICATION_WEBHOOK_URL: z.string().url().optional(),
  NOTIFICATION_EMAIL: z.string().email().optional(),
});
───────────────────────────────────────────────────────────────────

GLOBAL-DM-002 — COMPLETE DATABASE SCHEMA
File: specs/_global/data-model-complete.md
───────────────────────────────────────────────────────────────────

Tables
experiment_runs
SQL

CREATE TABLE experiment_runs (
  run_id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_label           TEXT NOT NULL,
  status              TEXT NOT NULL CHECK (status IN (
                        'INITIALISING','RUNNING','PAUSED','COMPLETED','FAILED'
                      )),
  node_id             UUID NOT NULL,            -- primary/initiating node

  -- Locked config (immutable after INITIALISING → RUNNING)
  metric_weights      JSONB NOT NULL,
  operating_conditions JSONB NOT NULL,
  solver_config       JSONB NOT NULL,
  mesh_config         JSONB NOT NULL,

  -- Progress
  total_experiments   INTEGER NOT NULL DEFAULT 0,
  kept_count          INTEGER NOT NULL DEFAULT 0,
  reverted_count      INTEGER NOT NULL DEFAULT 0,
  timeout_count       INTEGER NOT NULL DEFAULT 0,
  mesh_fail_count     INTEGER NOT NULL DEFAULT 0,

  -- Best-so-far
  best_M              DOUBLE PRECISION,
  best_experiment_n   INTEGER,
  best_geometry_sha256 TEXT,

  -- GEP state
  current_genome      JSONB,
  gep_generation      INTEGER NOT NULL DEFAULT 0,
  gep_in_progress     BOOLEAN NOT NULL DEFAULT FALSE,

  -- Swarm
  swarm_enabled       BOOLEAN NOT NULL DEFAULT FALSE,

  -- Baseline
  baseline_experiment_id UUID,

  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
experiments
SQL

CREATE TABLE experiments (
  experiment_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID NOT NULL REFERENCES experiment_runs(run_id),
  experiment_n        INTEGER NOT NULL,
  node_id             UUID NOT NULL,

  -- Mutation
  mutation_parameter_id TEXT NOT NULL,
  mutation_value_before DOUBLE PRECISION NOT NULL,
  mutation_value_after  DOUBLE PRECISION NOT NULL,
  mutation_delta        DOUBLE PRECISION NOT NULL,
  mutation_strategy     TEXT NOT NULL,
  topology_mutation     BOOLEAN NOT NULL DEFAULT FALSE,

  -- Geometry provenance
  geometry_sha256_before TEXT NOT NULL,
  geometry_sha256_after  TEXT,

  -- Stage results
  bounds_valid          BOOLEAN,
  mesh_quality_ok       BOOLEAN,
  mesh_cell_count       INTEGER,
  mesh_s3_key           TEXT,
  mesh_sha256           TEXT,
  solve_converged       BOOLEAN,
  solve_wall_time_s     DOUBLE PRECISION,
  solve_timeout         BOOLEAN NOT NULL DEFAULT FALSE,
  results_s3_key        TEXT,
  results_sha256        TEXT,

  -- Aerodynamic coefficients
  cl                    DOUBLE PRECISION,
  cd                    DOUBLE PRECISION,
  cmy                   DOUBLE PRECISION,
  ld_ratio              DOUBLE PRECISION,
  cd_wave               DOUBLE PRECISION,
  buffet_onset_alpha    DOUBLE PRECISION,

  -- Metric
  M                     DOUBLE PRECISION,
  M_best_at_time        DOUBLE PRECISION,
  delta_M               DOUBLE PRECISION,

  -- Decision
  status                TEXT NOT NULL CHECK (status IN (
                          'PENDING','BOUNDS_FAIL','MESH_FAIL',
                          'SOLVE_TIMEOUT','SOLVE_FAIL','EXTRACT_FAIL',
                          'REVERTED','KEPT','ABORTED'
                        )),
  git_commit_hash       TEXT,

  -- GEP context
  genome_generation     INTEGER,

  -- Timing
  stage_timings         JSONB,       -- {bounds_ms, mesh_ms, solve_ms, extract_ms, decision_ms}
  total_wall_time_s     DOUBLE PRECISION,

  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (run_id, experiment_n)      -- monotonicity + uniqueness constraint
);

-- Append-only enforcement via trigger
CREATE OR REPLACE FUNCTION prevent_experiment_update()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'UPDATE' THEN
    RAISE EXCEPTION 'experiments table is append-only';
  END IF;
  RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER experiments_append_only
  BEFORE UPDATE OR DELETE ON experiments
  FOR EACH ROW EXECUTE FUNCTION prevent_experiment_update();
knowledge_graph_entries
SQL

CREATE TABLE knowledge_graph_entries (
  entry_id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID REFERENCES experiment_runs(run_id),
  parameter_id        TEXT NOT NULL,
  experiment_n        INTEGER NOT NULL,
  node_id             UUID NOT NULL,
  delta_value         DOUBLE PRECISION NOT NULL,
  delta_M             DOUBLE PRECISION NOT NULL,
  outcome             TEXT NOT NULL CHECK (outcome IN ('KEPT','REVERTED',
                        'TIMEOUT','MESH_FAIL','BOUNDS_FAIL')),
  mesh_quality_ok     BOOLEAN,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX kg_param_run ON knowledge_graph_entries(parameter_id, run_id);
knowledge_graph_sensitivities
SQL

CREATE TABLE knowledge_graph_sensitivities (
  sensitivity_id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID REFERENCES experiment_runs(run_id),
  parameter_id        TEXT NOT NULL,
  sensitivity_score   DOUBLE PRECISION NOT NULL,
  direction           TEXT CHECK (direction IN ('positive','negative','neutral')),
  confidence          DOUBLE PRECISION NOT NULL,
  sample_count        INTEGER NOT NULL,
  dead_zone           BOOLEAN NOT NULL DEFAULT FALSE,
  hot_zone            BOOLEAN NOT NULL DEFAULT FALSE,
  computed_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (run_id, parameter_id)  -- upserted after each experiment
);
knowledge_graph_interactions
SQL

CREATE TABLE knowledge_graph_interactions (
  interaction_id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID REFERENCES experiment_runs(run_id),
  param_a             TEXT NOT NULL,
  param_b             TEXT NOT NULL,
  interaction_strength DOUBLE PRECISION NOT NULL,
  p_value             DOUBLE PRECISION NOT NULL,
  direction           TEXT CHECK (direction IN ('synergistic','antagonistic','neutral')),
  sample_count        INTEGER NOT NULL,
  discovered_at_exp   INTEGER NOT NULL,
  computed_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (run_id, param_a, param_b)
);
wiki_articles
SQL

CREATE TABLE wiki_articles (
  article_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID REFERENCES experiment_runs(run_id),
  title               TEXT NOT NULL,
  type                TEXT NOT NULL CHECK (type IN (
                        'SENSITIVITY','BATCH_SUMMARY',
                        'INTERACTION','TOPOLOGY'
                      )),
  experiment_range_start INTEGER,
  experiment_range_end   INTEGER,
  referenced_parameters  TEXT[],
  referenced_experiments INTEGER[],
  file_path           TEXT NOT NULL,
  body_markdown       TEXT NOT NULL,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
swarm_nodes
SQL

CREATE TABLE swarm_nodes (
  node_id             UUID PRIMARY KEY,
  run_id              UUID REFERENCES experiment_runs(run_id),
  hostname            TEXT NOT NULL,
  status              TEXT NOT NULL CHECK (status IN (
                        'ACTIVE','IDLE','SOLVING','GEP','STALE','OFFLINE'
                      )),
  current_region      TEXT,
  local_best_M        DOUBLE PRECISION,
  experiment_n        INTEGER,
  core_count          INTEGER,
  memory_gb           DOUBLE PRECISION,
  gpu_available       BOOLEAN DEFAULT FALSE,
  last_heartbeat      TIMESTAMPTZ NOT NULL DEFAULT now(),
  registered_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  deregistered_at     TIMESTAMPTZ
);
Database Indexes
SQL

-- High-frequency query paths
CREATE INDEX exp_run_n      ON experiments(run_id, experiment_n DESC);
CREATE INDEX exp_run_status ON experiments(run_id, status);
CREATE INDEX exp_node       ON experiments(node_id);
CREATE INDEX exp_kept       ON experiments(run_id) WHERE status = 'KEPT';
CREATE INDEX wiki_run_type  ON wiki_articles(run_id, type);
CREATE INDEX swarm_run      ON swarm_nodes(run_id, status);
───────────────────────────────────────────────────────────────────

GLOBAL-MSG-001 — COMPLETE MESSAGE PROTOCOL
File: specs/_global/messages.md
───────────────────────────────────────────────────────────────────

LoopMessage Base Type
TypeScript

interface LoopMessage {
  message_id:     string;       // UUID v4
  run_id:         string;       // UUID v4
  experiment_n:   number;       // monotonic integer
  node_id:        string;       // UUID v4
  timestamp:      string;       // ISO 8601 UTC
  message_type:   MessageType;
  stage:          Stage;
  payload:        Record<string, unknown>;
  error?:         { code: string; message: string; stack?: string };
}
MessageType Enum (complete)
TypeScript

enum MessageType {
  // Loop lifecycle
  LOOP_START                = 'LOOP_START',
  LOOP_PAUSE                = 'LOOP_PAUSE',
  LOOP_RESUME               = 'LOOP_RESUME',
  LOOP_HALT_DIRTY_TREE      = 'LOOP_HALT_DIRTY_TREE',
  LOOP_CONSECUTIVE_FAIL     = 'LOOP_CONSECUTIVE_FAIL',

  // Stage 0: Init
  RUN_INITIALISED           = 'RUN_INITIALISED',
  BASELINE_COMPLETE         = 'BASELINE_COMPLETE',
  BASELINE_FAILED           = 'BASELINE_FAILED',

  // Stage 1: Mutation
  MUTATION_PROPOSAL         = 'MUTATION_PROPOSAL',
  BOUNDS_VALID              = 'BOUNDS_VALID',
  BOUNDS_INVALID            = 'BOUNDS_INVALID',
  GEOMETRY_WRITTEN          = 'GEOMETRY_WRITTEN',
  MUTATION_SKIPPED_DEAD_ZONE = 'MUTATION_SKIPPED_DEAD_ZONE',
  MUTATION_SKIPPED_DUPLICATE = 'MUTATION_SKIPPED_DUPLICATE',

  // Stage 2: Mesh
  MESH_STARTED              = 'MESH_STARTED',
  MESH_COMPLETE             = 'MESH_COMPLETE',
  MESH_QUALITY_PASS         = 'MESH_QUALITY_PASS',
  MESH_QUALITY_FAIL         = 'MESH_QUALITY_FAIL',
  MESH_UPLOADED             = 'MESH_UPLOADED',

  // Stage 3: Solve
  SOLVE_STARTED             = 'SOLVE_STARTED',
  SOLVE_CONVERGED           = 'SOLVE_CONVERGED',
  SOLVE_UNCONVERGED         = 'SOLVE_UNCONVERGED',
  SOLVE_TIMEOUT             = 'SOLVE_TIMEOUT',
  SOLVE_FAILED              = 'SOLVE_FAILED',
  RESULTS_UPLOADED          = 'RESULTS_UPLOADED',

  // Stage 4: Extract + Metric
  EXTRACTION_COMPLETE       = 'EXTRACTION_COMPLETE',
  EXTRACTION_FAILED         = 'EXTRACTION_FAILED',
  METRICS_COMPUTED          = 'METRICS_COMPUTED',
  SANITY_WARNING            = 'SANITY_WARNING',

  // Stage 5: Keep/Revert
  EXPERIMENT_KEPT           = 'EXPERIMENT_KEPT',
  EXPERIMENT_REVERTED       = 'EXPERIMENT_REVERTED',
  EXPERIMENT_ABORTED        = 'EXPERIMENT_ABORTED',
  GLOBAL_BEST_UPDATED       = 'GLOBAL_BEST_UPDATED',

  // Stage 6: Wiki
  WIKI_COMPILE_STARTED      = 'WIKI_COMPILE_STARTED',
  WIKI_COMPILE_COMPLETE     = 'WIKI_COMPILE_COMPLETE',
  WIKI_ARTICLE_CREATED      = 'WIKI_ARTICLE_CREATED',

  // Stage 7: Knowledge Graph
  KG_UPDATE_COMPLETE        = 'KG_UPDATE_COMPLETE',
  KG_INTERACTION_DISCOVERED = 'KG_INTERACTION_DISCOVERED',
  KG_DEAD_ZONE_DECLARED     = 'KG_DEAD_ZONE_DECLARED',
  KG_HOT_ZONE_DECLARED      = 'KG_HOT_ZONE_DECLARED',

  // Stage 8: GEP
  GEP_EVOLUTION_START       = 'GEP_EVOLUTION_START',
  GEP_EVOLUTION_COMPLETE    = 'GEP_EVOLUTION_COMPLETE',
  GEP_EVOLUTION_FAILED      = 'GEP_EVOLUTION_FAILED',
  TOPOLOGY_ENABLED          = 'TOPOLOGY_ENABLED',

  // Stage 9: Swarm
  NODE_REGISTERED           = 'NODE_REGISTERED',
  NODE_HEARTBEAT            = 'NODE_HEARTBEAT',
  NODE_STALE                = 'NODE_STALE',
  NODE_DEREGISTERED         = 'NODE_DEREGISTERED',
  REGION_CLAIMED            = 'REGION_CLAIMED',
  REGION_RELEASED           = 'REGION_RELEASED',
  REGION_CONFLICT_RESOLVED  = 'REGION_CONFLICT_RESOLVED',
  SWARM_OFFLINE_MODE        = 'SWARM_OFFLINE_MODE',
  SWARM_RECONNECTED         = 'SWARM_RECONNECTED',
}
Stage Enum
TypeScript

enum Stage {
  INIT       = 0,
  MUTATION   = 1,
  MESH       = 2,
  SOLVE      = 3,
  EXTRACT    = 4,
  DECISION   = 5,
  WIKI       = 6,
  KG         = 7,
  GEP        = 8,
  SWARM      = 9,
}
BullMQ Queue Layout
text

Queue: aeroloop-{run_id}-core
  Jobs: MUTATION, MESH, SOLVE, EXTRACT, DECISION (serial, per experiment)

Queue: aeroloop-{run_id}-background
  Jobs: KG_UPDATE, WIKI_COMPILE, SWARM_SYNC (parallel, non-blocking)

Queue: aeroloop-{run_id}-gep
  Jobs: GEP_EVOLUTION (serial, blocking loop during execution)

Queue: aeroloop-swarm-heartbeat
  Jobs: NODE_HEARTBEAT (repeated, every 30s per node)
Job options (core queue):

TypeScript

{
  attempts: 1,           // never retry core jobs; always revert and move on
  timeout: (KILL_TIMER_S + 60) * 1000,  // application-level backstop
  removeOnComplete: 200, // keep last 200 completed job records
  removeOnFail: 500,
}
───────────────────────────────────────────────────────────────────

GLOBAL-COMMIT-001 — GIT COMMIT MESSAGE SPECIFICATION
File: specs/_global/commit-format.md
───────────────────────────────────────────────────────────────────

All loop-generated commits MUST match the following format exactly. The commit-msg git hook enforces this for commits authored by GIT_AUTHOR_NAME (AeroLoopBot).

KEEP commit
text

exp({experiment_n}): keep {parameter_id} {before} → {after} | M={M_new:.6f} Δ={delta_M:+.6f}

run_id: {run_id}
node_id: {node_id}
geometry_sha256_before: {sha256_before}
geometry_sha256_after:  {sha256_after}
mesh_sha256: {mesh_sha256}
results_sha256: {results_sha256}
cl={cl:.4f} cd={cd:.4f} ld={ld:.4f}
solve_wall_time_s={solve_wall_time_s:.1f}
strategy={strategy}
genome_generation={genome_generation}
Example:

text

exp(0144): keep taper_ratio 0.410 → 0.380 | M=0.882100 Δ=+0.003200

run_id: a1b2c3d4-...
node_id: e5f6g7h8-...
...
GEP genome commit
text

chore(gep): generation {N} evolved at exp {experiment_n}

run_id: {run_id}
fitness_before: {fitness:.4f}
fitness_after:  {fitness:.4f}
strategy_before: {strategy}
strategy_after:  {strategy}
population_size: 12
Wiki compile commit
text

docs(wiki): batch {start_n}–{end_n} compiled | {article_count} articles

run_id: {run_id}
articles: [{list of article titles}]
Hook implementation (hooks/commit-msg):
Bash

#!/usr/bin/env bash
# Only enforce format for AeroLoopBot commits
if [ "$GIT_AUTHOR_NAME" = "AeroLoopBot" ]; then
  COMMIT_MSG=$(cat "$1")
  if ! echo "$COMMIT_MSG" | grep -qE \
    '^(exp\([0-9]+\):|chore\(gep\):|docs\(wiki\):)'; then
    echo "ERROR: AeroLoopBot commit message does not match spec format."
    exit 1
  fi
fi
exit 0
───────────────────────────────────────────────────────────────────

GLOBAL-METRIC-001 — COMPOSITE METRIC SPECIFICATION (COMPLETE)
File: specs/_global/metric.md
───────────────────────────────────────────────────────────────────

Metric Definition
The composite metric M is the single scalar used for all keep/revert decisions. It is defined once per run in ExperimentRun.metric_weights and is immutable for the duration of the run.

Base formula (with buffet data available)
text

M = w_ld  × norm(L/D)
  + w_buffet × norm(buffet_margin_deg)
  + w_wave   × norm(1 - Cd_wave / Cd_total)
Where norm(x) = x / x_baseline (ratio to baseline value). If baseline value is 0 for any term, that term is excluded and weights are renormalised to sum to 1.0.

Fallback (no buffet data — alpha sweep not run)
w_buffet is distributed proportionally to w_ld and w_wave:

text

w_ld_adj  = w_ld  + w_buffet × (w_ld  / (w_ld + w_wave))
w_wave_adj = w_wave + w_buffet × (w_wave / (w_ld + w_wave))
M = w_ld_adj × norm(L/D) + w_wave_adj × norm(1 - Cd_wave / Cd_total)
Weight constraints (enforced at run creation):
All weights ≥ 0.0
Sum of weights = 1.0 ± 0.001
w_ld ≥ 0.3 (L/D must always be a meaningful component)
Wave drag computation
Primary method (SU2 far-field decomposition, if available in solver output):

text

Cd_wave = Cd_total - Cd_induced - Cd_viscous
Fallback (if far-field decomposition unavailable):

text

Cd_induced ≈ CL² / (π × AR × e)     [e = Oswald efficiency, default 0.85]
Cd_viscous ≈ Cf × (1 + 1.2×(t/c) + 100×(t/c)⁴) × S_wet/S_ref
Cd_wave = max(0, Cd_total - Cd_induced - Cd_viscous)
Sanity bounds (warn, do not halt):
Coefficient	Warn if outside
CL	[-0.5, 3.0]
CD	[0.001, 0.5]
L/D	[0, 80]
CMy	[-1.0, 1.0]
Cd_wave	< 0 (clamp to 0)
Metric version
Each ExperimentRun records metric_version = "1.0". Any change to the metric formula requires a version bump and a new run; never retroactively re-score old experiments.

───────────────────────────────────────────────────────────────────

GLOBAL-MESH-QUALITY-001 — MESH QUALITY THRESHOLDS SPECIFICATION
File: specs/_global/mesh-quality.md
───────────────────────────────────────────────────────────────────

Quality Gates (all must pass for MESH_QUALITY_PASS)
Metric	Threshold	Fail condition
Min orthogonal quality	≥ 0.15	any cell below 0.15
Max skewness	≤ 0.85	any cell above 0.85
y+ (first cell, 95th %ile)	≤ 5.0	95th percentile > 5
y+ target (mean)	0.5 – 2.0	mean outside range
Cell count	within ±30% of target	outside range
Negative volume cells	0	any < 0
Max aspect ratio	≤ 5000	any cell above 5000
Mesh Level Targets
Level	Target cell count	BL layers	Growth rate
coarse	500k – 1.5M	20	1.20
medium	1.5M – 4M	30	1.15
fine	4M – 10M	40	1.10
Quality Report Format
MeshQualityAgent must write a quality report to .aeroloop/mesh_quality_exp_{N}.json:

JSON

{
  "experiment_n": 144,
  "passed": true,
  "metrics": {
    "min_orthogonal_quality": 0.31,
    "max_skewness": 0.71,
    "y_plus_95th": 1.8,
    "y_plus_mean": 0.94,
    "cell_count": 2140000,
    "cell_count_target": 2000000,
    "negative_volume_cells": 0,
    "max_aspect_ratio": 1240
  },
  "failures": []
}
───────────────────────────────────────────────────────────────────

OPERATOR-001 — OPERATOR RUNBOOK
File: specs/operator/runbook.md
───────────────────────────────────────────────────────────────────

Starting a run (single node)
Bash

# 1. Ensure git working tree is clean
git status   # must show "nothing to commit"

# 2. Validate environment
pnpm cli status --check-env

# 3. Start the loop
pnpm cli start \
  --run-label "naca0012-sweep-001" \
  --mach 0.78 \
  --reynolds 4.5e7 \
  --alpha 2.5 \
  --w-ld 0.5 \
  --w-buffet 0.3 \
  --w-wave 0.2 \
  --kill-timer 480

# 4. Monitor
open http://localhost:3000   # dashboard
pnpm cli status              # CLI status summary
Starting a swarm run (N nodes)
Bash

# On coordinator node (also runs the API server):
pnpm cli start --swarm --nodes 4 --run-label "sweep-swarm-001" [options]

# On each worker node (different machines or containers):
SWARM_ENABLED=true \
NODE_COORDINATOR_URL=http://coordinator:4000 \
pnpm cli start --join-run {run_id}
Pausing a run
Bash

pnpm cli pause --run-id {run_id} [--node-id {node_id}|--all]
# Loop pauses after current experiment completes (not mid-stage)
Resuming a run
Bash

pnpm cli resume --run-id {run_id} [--node-id {node_id}|--all]
Aborting the current experiment
Bash

pnpm cli abort-experiment --run-id {run_id} --node-id {node_id}
# Kill timer fires immediately; geometry reverted; loop continues
Exporting results
Bash

# Export all KEPT experiments as CSV
pnpm cli export --run-id {run_id} --format csv --output ./results.csv

# Export full experiment history as JSON
pnpm cli export --run-id {run_id} --format json --output ./full_history.json

# Export knowledge graph summary
pnpm cli export --run-id {run_id} --type kg --output ./kg_summary.json
Recovering from a dirty working tree halt
Bash

# The loop halts and emits LOOP_HALT_DIRTY_TREE if it detects uncommitted changes
# to any file other than wing.geo.
# Resolution:
git stash         # or git checkout -- .
git status        # must be clean
pnpm cli resume --run-id {run_id}
Recovering from coordinator loss (swarm)
Bash

# Nodes enter OFFLINE_MODE automatically.
# Restart coordinator:
docker compose restart api
# Nodes detect reconnect via next heartbeat cycle (≤ 30s) and re-register.
# Check accumulated offline records synced:
pnpm cli status --run-id {run_id} --check-sync
Monitoring key metrics (CLI)
Bash

pnpm cli status --run-id {run_id}
# Output:
# Run:         naca0012-sweep-001 (RUNNING)
# Experiments: 144 total | 41 kept (28.5%) | 3 timeouts | 2 mesh fails
# Best M:      0.8821 @ exp-138
# Rate:        8.2 experiments/hour (rolling 1h)
# Consecutive fails: 0
# GEP:         generation 0 | next at exp 500
# Active nodes: 1
───────────────────────────────────────────────────────────────────

GLOBAL-TEST-001 — TESTING PHILOSOPHY & PATTERNS
File: specs/_global/testing.md
───────────────────────────────────────────────────────────────────

Testing principles
Unit tests own correctness of pure logic. All pure functions (metric computation, bounds checking, genome operators, wave drag formula, buffet detection) must have exhaustive unit test coverage with known-good values from reference aerodynamics data.

Integration tests own the pipeline contracts. Each stage boundary (e.g., mesh → quality → solver) is tested end-to-end with real GMSH and mocked SU2 (pre-recorded output), or real SU2 on CI runners with a budget geometry (fast-solving airfoil).

Git integrity tests are non-negotiable. Every test that touches keep/revert must assert: SHA256 before == SHA256 after revert, and commit message format validity on keep.

No test may use sleep() or wall-clock assertions for correctness. Use injected timers / fake clocks for kill-timer tests.

Test data is fixed, versioned, and committed. Reference meshes and solver outputs live in packages/extraction/fixtures/ and packages/meshing/fixtures/. Never regenerate them silently.

Naming convention
text

packages/{name}/tests/
  unit/
    {module}.test.ts
  integration/
    {module}.integration.test.ts
apps/{name}/tests/
  e2e/
    {scenario}.e2e.test.ts
Test environment
Vitest as the test runner (all packages and apps).
@testcontainers/postgresql and @testcontainers/redis for integration tests that need real DB/Redis.
S3 mocked via minio testcontainer or @aws-sdk/client-s3 mock.
SU2 mocked via MockSU2Runner injectable (same interface as real SU2Runner; returns pre-recorded fixture outputs).
GMSH runs real in integration tests; coarse mesh only for speed.
───────────────────────────────────────────────────────────────────

GLOBAL-LOGGING-001 — STRUCTURED LOGGING SPECIFICATION
File: specs/_global/logging.md
───────────────────────────────────────────────────────────────────

All log output is structured JSON (using pino). No plain-text log statements allowed in production code.

Required fields on every log line
JSON

{
  "level": "info",
  "time": "2025-01-15T14:32:01.123Z",
  "service": "loop|api|dashboard|cli",
  "run_id": "uuid-v4",
  "experiment_n": 144,
  "node_id": "uuid-v4",
  "stage": 2,
  "message": "Mesh quality check passed",
  "...additional context fields..."
}
Required log events (loop process)
Event	Level	Required context fields
Experiment start	info	experiment_n, mutation_parameter_id, delta
Bounds check result	info	valid, parameter_id, value, bounds
Mesh started	info	level, cell_count_target
Mesh quality result	info	passed, all quality metrics
Solve started	info	config_hash, mpi_ranks, kill_timer_s
Solve timeout	warn	wall_time_s, kill_timer_s
Solve converged	info	wall_time_s, final_residual, iterations
Metric computed	info	M, cl, cd, ld, cd_wave, buffet_onset_alpha
KEEP decision	info	M_new, M_best, delta_M, git_commit_hash
REVERT decision	info	M_new, M_best, delta_M
Consecutive fail count	warn	consecutive_fail_count, threshold
Dirty tree halt	error	dirty_files: string[]
GEP start	info	generation, experiment_n, genome_summary
KG interaction discovered	info	param_a, param_b, p_value, strength
Dead zone declared	warn	parameter_id, attempts, region
Log retention
Development: stdout only
Production: ship to configured log aggregator (Loki/ELK); retain 90 days
Experiment decision logs (KEPT/REVERTED) are also appended to .aeroloop/decision_log.jsonl locally (human-readable audit trail)
───────────────────────────────────────────────────────────────────

IMPLEMENTATION ROADMAP
File: program.md (completion)
───────────────────────────────────────────────────────────────────

Phase 0 — Foundation (Week 1–2)
Goal: working monorepo skeleton with all shared types, DB schema, and git integrity primitives.

 pnpm workspace + Turborepo setup
 packages/types — all Zod schemas, compiled and exported
 packages/db — Drizzle schema, migrations, append-only trigger
 packages/redis — client + key helpers + SETNX claim helpers
 packages/git-ops — clean-check, keep, revert (100% tested)
 packages/geometry — parameter registry, bounds checker, wing.geo parser (95% tested with full parameter set)
 hooks/pre-commit + hooks/commit-msg — installed and tested
 infra/docker-compose.yml — postgres, redis, minio running
 CI pipeline (typecheck + lint + unit tests passing)
Milestone gate: packages/git-ops integration test passes (clean-check + keep + revert cycle on real git repo with wing.geo).

Phase 1 — Core Pipeline (Week 3–5)
Goal: single experiment runs end-to-end (Stages 0–5) with real GMSH and mocked SU2.

 packages/meshing — auto_mesh.py wrapper, quality checker, S3 upload with SHA256 (medium level only initially)
 packages/solver — SU2Runner interface, MockSU2Runner, dual kill-timer implementation (OS + BullMQ), convergence check
 packages/extraction — polar extractor, wave drag, metric computer (95% unit tested with reference aerodynamic data)
 apps/loop — Stage 0–5 orchestrator, BullMQ core queue wired
 apps/api — /runs, /experiments endpoints (POST + GET)
 Integration tests IT-001 through IT-004 passing
 Stage 0 baseline run completes end-to-end (mocked SU2)
Milestone gate: 50-experiment run completes autonomously overnight with ≥1 KEEP, zero git integrity violations, and full append-only experiment history in DB.

Phase 2 — Learning Layer (Week 6–8)
Goal: knowledge graph, wiki, and informed mutations working.

 packages/knowledge-graph — updater, impact detector, interaction discovery, dead/hot zone tracking
 packages/wiki — sensitivity + batch + interaction article compilers (enforce "specific numbers + experiment refs" rule)
 apps/api — /knowledge-graph and /wiki endpoints
 apps/loop — Stages 6–7 wired into background queue
 GeometryMutationAgent reads KG summary + wiki before proposing
 Anti-repetition check (last 10 experiments) implemented
 Integration test: KG interaction discovered at p < 0.05
Milestone gate: 200-experiment run shows decreasing mutation attempts in declared dead zones (verifiable from experiment log).

Phase 3 — Adaptive Strategy (Week 9–10)
Goal: GEP evolver working; genome evolves after 500 experiments.

 packages/gep — genome schema, fitness (history-only), mutation, crossover, selection (population = 12, elites = 3)
 apps/loop — Stage 8 wired; loop pauses correctly at n%500==0
 GEP integration test IT-005 passing
 Topology macro-mutation gated correctly by topology_probability
 Operator notification on GEP trigger
Milestone gate: IT-005 passes; genome file committed with correct message format; loop resumes without manual intervention.

Phase 4 — Swarm (Week 11–13)
Goal: multi-node swarm running with non-collision and shared learning.

 apps/api — /swarm endpoints + SSE stream
 apps/loop — Stage 9 swarm guard wired; region claim/release
 apps/dashboard — React dashboard with SSE consumer
 apps/cli — all operator commands implemented
 OFFLINE_MODE + reconnect implemented
 Integration test IT-006 passing (4-node non-collision)
 K8s manifests (loop StatefulSet, api Deployment, HPA)
Milestone gate: 4-node swarm completes 500 experiments with zero git conflicts and all experiment records reconciled in shared DB.

Phase 5 — Production Hardening (Week 14–16)
Goal: production-ready system; real SU2 on real geometry.

 Replace MockSU2Runner with real SU2 runner in all integration tests (using NACA 0012 airfoil as budget geometry — fast solve, known polars)
 Full mesh level support (coarse / medium / fine)
 Alpha sweep for buffet onset fully wired
 Surrogate model interface (after 50 experiments) implemented (even if initial model is a simple linear regression)
 Terraform (or k8s Helm chart) for cloud deployment
 Release pipeline wired; v1.0.0 release tagged
 Full operator runbook validated by running 1000-experiment unattended overnight test on real wing geometry
Milestone gate: 1000-experiment autonomous run on real wing.geo shows monotonically non-decreasing best_M (no regressions), with < 5% experiment loss rate (timeout + mesh fail combined).

───────────────────────────────────────────────────────────────────

OPEN ITEMS & KNOWN GAPS (as of spec v1.0)
File: specs/_global/open-items.md
───────────────────────────────────────────────────────────────────

These items are known to be underspecified and must be resolved before Phase 5 production hardening.

ID	Area	Gap description	Proposed resolution
OI-001	Surrogate model	Interface defined but algorithm not specified.	Use Gaussian Process Regression (GPR) via scikit-learn;
retrain every 50 experiments; use only as advisory input
to GeometryMutationAgent, not as a replacement for CFD.
OI-002	Multi-objective	Current metric collapses to scalar M. Pareto front	Post-v1.0: add Pareto mode flag; store CL/CD/buffet as
exploration not supported.	separate objectives; use NSGA-II as optional GEP strategy.
OI-003	Turbulence model	SU2 config template uses SA model by default. k-ω SST	Add turbulence_model to solver_config; validate against
may be more accurate for high-AoA buffet prediction.	reference polars for NACA 0012 at Mach 0.78.
OI-004	3D vs 2D geometry	Spec assumes 3D wing.geo. No 2D airfoil mode defined.	Add `geometry_mode: '2D'
single-section GMSH template and different SU2 BC.
OI-005	Authentication	JWT auth specified but no user management defined.	For v1.0: single shared secret via env var. Post-v1.0:
add user table + role (admin / read-only).
OI-006	Knowledge graph merge	Last-write-wins for swarm KG merge may miss nuance	Post-v1.0: implement Bayesian update (weighted by
when two nodes explore the same parameter from	sample count) instead of last-write-wins.
different directions simultaneously.	
OI-007	Cost model	No compute-cost tracking. Expensive swarm runs have	Add cost_usd field to ExperimentRecord; compute from
no budget governor.	solve_wall_time_s × node_cost_per_hour (env var).
OI-008	Wing parametrisation	wing.geo format assumed but not formally specified.	Publish GLOBAL-GEOM-002 wing.geo format spec defining
GMSH .geo macro syntax, variable naming, and section IDs.
OI-009	Restart / warm-start	Cold restart from mid-run defined only at loop level.	Specify explicit checkpoint file .aeroloop/checkpoint.json
Solver restart (SU2 restart files) not yet specified.	carrying: last experiment_n, current genome, best SHA256.
OI-010	Dashboard auth	Dashboard reads SSE stream; no auth in dev mode.	Add optional DASHBOARD_READ_TOKEN env var for staging.
───────────────────────────────────────────────────────────────────

SPEC CHANGELOG
File: specs/_global/changelog.md
───────────────────────────────────────────────────────────────────

Version	Date	Author	Changes
0.1	2025-01-01	knarayanareddy	Initial draft: Stages 0–8 core pipeline
0.2	2025-01-08	knarayanareddy	Added GEP stage, constitutional law, message proto
0.3	2025-01-12	knarayanareddy	Added knowledge graph, wiki, swarm constitutional
1.0	2025-01-15	knarayanareddy	Completed: Stage 9, API contracts, DB schema,
infra spec, metric spec, mesh quality spec,
commit format, logging, testing, operator runbook,
implementation roadmap, open items.
<!-- END OF Aeroloopspec.md v1.0 -->
text


---

## What was completed & why each section belongs here

Here is a quick summary of every section added and the design rationale behind each:

| Section | Why it was needed |
|---|---|
| **Stage 8 Acceptance Criteria (AC-GEP-001–006)** | The original spec had requirements but no testable acceptance criteria — without ACs, you can't call Stage 8 "done" |
| **Stage 9 — Swarm Coordination (full)** | Only referenced in constitutional law; the actual FR, region model, shared sync, failure modes, and ACs were entirely absent |
| **GLOBAL-API-001 — API Contracts** | Referenced by nearly every stage agent (`GET /knowledge-graph/summary`, `POST /experiments`, etc.) but never defined; without this no agent can be implemented correctly |
| **GLOBAL-INFRA-001 — Infra & CI/CD** | The monorepo layout was sketched; the complete tree, Docker Compose, CI job specs, integration test specs, and release pipeline were missing — all required to build the system |
| **GLOBAL-ENV-001 — Environment Config** | Environment variables referenced throughout (kill timer, S3, SU2 path, etc.) but never consolidated into a validated schema; without this startup validation is impossible |
| **GLOBAL-DM-002 — Complete DB Schema** | Stage specs referenced DB fields; only partial schema was implied — the full SQL DDL, append-only trigger, indexes, and all tables (including KG, wiki, swarm) were missing |
| **GLOBAL-MSG-001 — Complete Message Protocol** | Partial enum existed; the complete `MessageType` enum covering all 9 stages, the `Stage` enum, and BullMQ queue layout were needed for the orchestrator state machine |
| **GLOBAL-COMMIT-001 — Commit Format** | Referenced as a standard format throughout but never formally specified; the hook implementation enforcing it was absent |
| **GLOBAL-METRIC-001 — Metric Spec** | The formula was partially described in Stage 4; a canonical, versioned, complete definition (including fallback, wave drag computation, sanity bounds, weight constraints) is needed as a single source of truth |
| **GLOBAL-MESH-QUALITY-001 — Mesh Quality** | Thresholds were referenced but never formally tabulated in a single canonical spec |
| **OPERATOR-001 — Runbook** | Essential for unattended operation; completely absent from the original |
| **GLOBAL-TEST-001 — Testing Philosophy** | Needed to enforce consistency across packages (Vitest, testcontainers, fixture policy, naming convention) |
| **GLOBAL-LOGGING-001 — Logging Spec** | Pino structured logging was implied; field requirements and required event table were needed for observability |
| **Implementation Roadmap (program.md)** | Phases 0–5 with milestone gates translate the spec into an actionable build order with clear "done" criteria per phase |
| **Open Items** | Honest spec engineering requires cataloguing known gaps so they aren't silently ignored during implementation |
| **Changelog** | Required for any versioned spec document |



