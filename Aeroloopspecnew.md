Part 1 of 3 — covering the constitutional layer, agent registry, programme overview, global architecture, and data model.

Markdown

# Aeroloopspec.md
# AeroLoop — Autonomous CFD Wing Optimisation System
# Specification Bundle v1.0
# ─────────────────────────────────────────────────
# This file is the single source of truth for the AeroLoop system.
# It is organised as a concatenation of virtual spec files that would
# normally live across the monorepo. The boundary of each virtual file
# is marked with a header comment.
#
# TABLE OF CONTENTS
# ─────────────────
# PART 1
#   CLAUDE.md          — Constitutional law (inviolable rules)
#   AGENTS.md          — Agent registry + message protocol
#   program.md         — Programme overview + implementation roadmap
#   GLOBAL-ARCH-001    — Global architecture + data flow
#   GLOBAL-DM-001      — Complete database schema
#   GLOBAL-GEOM-001    — Geometry parameter registry
#
# PART 2
#   GLOBAL-API-001     — API contract specification
#   GLOBAL-MSG-001     — Complete message protocol
#   GLOBAL-METRIC-001  — Composite metric specification
#   GLOBAL-MESH-001    — Mesh quality thresholds
#   GLOBAL-ENV-001     — Environment & configuration
#   GLOBAL-COMMIT-001  — Git commit message format
#   GLOBAL-LOG-001     — Structured logging specification
#   GLOBAL-TEST-001    — Testing philosophy & patterns
#   Stage 0            — Initialisation
#   Stage 1            — Geometry mutation
#   Stage 2            — Mesh generation
#   Stage 3            — CFD solve
#   Stage 4            — Results extraction + metric
#   Stage 5            — Keep / Revert decision
#
# PART 3
#   Stage 6            — Wiki compilation
#   Stage 7            — Knowledge graph
#   Stage 8            — GEP genome evolver
#   Stage 9            — Swarm coordination
#   GLOBAL-INFRA-001   — Infrastructure & CI/CD
#   OPERATOR-001       — Operator runbook
#   GLOBAL-OPEN-001    — Open items & known gaps
#   GLOBAL-CHANGELOG   — Spec changelog
# ─────────────────────────────────────────────────────────────────────


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: CLAUDE.md
CONSTITUTIONAL LAW — INVIOLABLE RULES
═══════════════════════════════════════════════════════════════════════

These rules are constitutional. No agent, no stage requirement, and no
operator command may override them. If any rule conflicts with a stage
spec, the rule in CLAUDE.md wins. Violations that are detectable at
runtime must halt the loop immediately and emit a LOOP_HALT event.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-001: THE ONE MUTABLE FILE LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
During the optimisation loop, exactly ONE file may ever be modified:

    geometry/wing.geo

No other file in the working tree may be written, renamed, or deleted
by any loop process. The ONLY agent permitted to write wing.geo is the
GeometryMutationAgent. All other agents are read-only with respect to
the filesystem.

Enforcement: the pre-commit git hook rejects any loop commit that
touches files other than geometry/wing.geo. The hook checks the
GIT_AUTHOR_NAME env var and applies enforcement only to AeroLoopBot
commits, leaving developer commits unrestricted.

Rationale: atomic, auditable experiments. If anything other than
wing.geo mutates, an observed metric change cannot be attributed to
the geometry change alone.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-002: THE KILL TIMER LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Every CFD solve MUST be bounded by a hard wall-time limit. The limit
is set by the environment variable KILL_TIMER_S (default: 480 seconds).

Enforcement is DUAL-LAYER — both must be active simultaneously:
  Layer 1 (OS):          timeout ${KILL_TIMER_S} mpirun SU2_CFD ...
  Layer 2 (queue):       BullMQ job timeout = (KILL_TIMER_S + 60) × 1000 ms

Layer 2 is the backstop; Layer 1 is the primary. Neither alone is
sufficient. If Layer 1 fires: geometry is reverted, experiment is
recorded as SOLVE_TIMEOUT, loop continues. If Layer 2 fires first
(Layer 1 failed for any reason): same outcome.

No exceptions. No configuration knob disables both layers simultaneously.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-003: THE GIT INTEGRITY LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Every experiment produces EXACTLY ONE of two git outcomes:

  KEEP (metric improved):
    git add geometry/wing.geo
    git commit -m "<must exactly match RULE-060 format>"
    → one commit, one kept geometry

  REVERT (metric did not improve, timeout, mesh fail, solve fail,
          extraction fail, or bounds fail):
    git checkout geometry/wing.geo
    → no commit; working tree restored to HEAD

There is NO third outcome. There is no "git revert HEAD" (which creates
a revert commit). There is no leaving the tree dirty. The working tree
MUST be clean at the START of every experiment. A dirty working tree
at experiment start is an immediate LOOP_HALT_DIRTY_TREE event.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-004: THE PHYSICAL BOUNDS LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Any mutation that produces a parameter value outside the hard bounds
defined in GLOBAL-GEOM-001 is REJECTED before wing.geo is written.
Rejection triggers an immediate REVERT (no file has been written, so
this is a no-op restore) and the experiment is recorded as BOUNDS_FAIL.
The bounds are not configurable per-run; they are constants in the
parameter registry.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-005: THE RESULT PROVENANCE LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
A result does not exist unless:
  (a) it has an ExperimentRecord row in PostgreSQL, AND
  (b) the solver output files referenced in that record exist in S3,
      AND their SHA256 checksums match the values stored in the record.

Any result used for a keep/revert decision that cannot satisfy (a) and
(b) is treated as if the solve failed. The experiment is reverted.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-006: THE SWARM NON-COLLISION LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
In swarm mode, nodes MUST claim non-overlapping design regions via
atomic Redis SETNX before writing wing.geo. A node that fails to
acquire its region claim MUST NOT write wing.geo. Region claims use
TTL (default 600 s) and are renewed on every heartbeat.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-007: THE LOOP STRUCTURE LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
The optimisation loop structure is inviolable. Every experiment MUST
traverse stages in this exact order:

  0-Init (once per run) →
  1-Mutate → 2-Mesh → 3-Solve → 4-Extract → 5-Decide →
  [background: 6-Wiki, 7-KG, 8-GEP(every 500), 9-Swarm]
  → repeat from 1

No stage may be skipped except by the defined failure short-circuits
(which all terminate at 5-Decide with a REVERT outcome).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-008: THE BEAT-THE-BEST LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Keep/revert is decided against M_best (the best metric ever recorded
in this run across all nodes), NOT against the previous experiment's
metric. Formally:

  KEEP  iff  M_new > M_best
  REVERT     otherwise (including M_new == M_best)

M_best is initialised from the baseline experiment (exp-0).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-009: THE CONSECUTIVE FAILURE LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
If CONSECUTIVE_FAIL_HALT (default: 3) experiments in a row result in
any non-KEPT outcome (REVERT, TIMEOUT, MESH_FAIL, etc.), the loop
PAUSES and emits a LOOP_CONSECUTIVE_FAIL event. The operator must
manually resume. This prevents runaway behaviour on a stuck design space.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-010: THE SINGLE SOLVE PER NODE LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
A node MUST NOT run more than one SU2 solve simultaneously. An
exclusive Redis lock (key: aeroloop:{run_id}:node:{node_id}:solve_lock)
must be held for the entire duration of the solve. Attempting to
acquire a second solve lock on the same node halts the second job.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-011: THE APPEND-ONLY EXPERIMENT RECORD LAW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Experiment records in PostgreSQL are APPEND-ONLY. No UPDATE or DELETE
is permitted on the experiments table. This is enforced by a database
trigger (see GLOBAL-DM-001). Any code path that attempts an UPDATE is
a bug. Stage result fields are populated in a single INSERT at the end
of Stage 5, not in incremental updates through the pipeline.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-012: THE DEPENDENCY CHAIN LAW (package build order)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Packages depend on each other strictly in this order (no circular deps):

  types → db → redis → s3 → geometry → meshing → solver →
  extraction → git-ops → knowledge-graph → surrogate → gep → wiki

Apps depend on any packages but not on other apps.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-060: GIT COMMIT FORMAT LAW (KEEP commits only)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Every KEEP commit MUST use EXACTLY this message format (no variation):

  exp({N}): M={M_new:.4f} delta={delta_M:+.4f} L/D={ld:.2f} [KEEP]
  mutation: {parameter_id}={old_val}→{new_val}
  mesh: {cell_count} cells | solve: {wall_time_s:.1f}s | convg: {residual_orders:.1f}ord

  run_id: {run_id}
  node_id: {node_id}
  geometry_sha256_before: {sha256_before}
  geometry_sha256_after:  {sha256_after}
  mesh_sha256: {mesh_sha256}
  results_sha256: {results_sha256}
  cl={cl:.4f} cd={cd:.4f}
  strategy={strategy}
  genome_generation={genome_generation}

The commit-msg git hook enforces this format for all AeroLoopBot
commits and rejects any commit that does not match.

REVERT outcomes produce NO commit. Only git checkout geometry/wing.geo.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RULE-061: GEP AND WIKI COMMIT FORMATS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GEP genome commit:
  chore(gep): generation {N} evolved at exp {experiment_n}
  run_id: {run_id}
  fitness_before: {f:.4f} → fitness_after: {f:.4f}
  strategy_before: {s} → strategy_after: {s}
  population_size: 16

Wiki compile commit:
  docs(wiki): batch {start_n}–{end_n} compiled | {n} articles
  run_id: {run_id}


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: AGENTS.md
AGENT REGISTRY + MESSAGE PROTOCOL
═══════════════════════════════════════════════════════════════════════

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AGENT REGISTRY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Orchestrators
─────────────
LoopOrchestrator      Stage state machine. Owns the core BullMQ queue.
                      Advances pipeline stage on each message received.
                      Emits LOOP_START on first run, LOOP_PAUSE on halt.

SwarmOrchestrator     Coordinates multi-node region claims and heartbeats.
                      Runs in apps/loop as a background process alongside
                      LoopOrchestrator when SWARM_ENABLED=true.

GEPOrchestrator       Manages genome evolution lifecycle. Triggered by
                      LoopOrchestrator at experiment_n % 500 === 0.
                      Blocks the core queue during evolution.

Core Per-Experiment Agents (run once per experiment, in order)
──────────────────────────────────────────────────────────────
GeometryMutationAgent Stage 1. Proposes and writes exactly one mutation
                      to geometry/wing.geo. THE ONLY AGENT ALLOWED TO
                      WRITE wing.geo. Tools: read_file, write_file (wing.geo
                      only), detect_impact (KG API), read_kg_summary,
                      read_wiki, read_genome.

BoundsCheckAgent      Stage 1 (post-mutation). Validates proposed parameter
                      value against GLOBAL-GEOM-001 registry. Pure function;
                      no I/O. Tools: read_parameter_registry.

MeshAgent             Stage 2. Runs GMSH pipeline via auto_mesh.py.
                      Uploads mesh to S3. Tools: run_gmsh, upload_s3,
                      sha256_file.

MeshQualityAgent      Stage 2 (post-mesh). Validates mesh against
                      GLOBAL-MESH-001 thresholds. Tools: read_mesh_metrics.

SolverAgent           Stage 3. Generates SU2 config, runs solve with dual
                      kill timers, uploads results to S3. Tools: run_su2,
                      upload_s3, sha256_file, acquire_solve_lock,
                      release_solve_lock.

ExtractionAgent       Stage 4. Parses SU2 outputs; computes CL, CD, CMy,
                      L/D, Cd_wave, buffet onset. Tools: read_s3,
                      parse_csv, compute_wave_drag, compute_buffet.

MetricAgent           Stage 4 (post-extraction). Computes composite M.
                      Pure function. Tools: none (receives coefficients).

KeepRevertAgent       Stage 5. Applies RULE-003 and RULE-008. Writes
                      ExperimentRecord to DB. Tools: git_commit,
                      git_checkout_restore, write_db_experiment_record,
                      update_run_best_M, update_redis_global_best.

Background Agents (non-blocking, run in background queue)
──────────────────────────────────────────────────────────
WikiCompilerAgent     Stage 6. Triggered every 10 experiments. Compiles
                      sensitivity, batch summary, and interaction articles.
                      Tools: read_db_experiments, write_wiki_article,
                      write_db_wiki_record, git_commit_wiki.

KnowledgeGraphAgent   Stage 7. Triggered after every experiment.
                      Updates sensitivity distributions, discovers
                      interactions, tracks dead/hot zones. Tools:
                      read_db_experiments, write_db_kg_entry,
                      upsert_db_sensitivity, upsert_db_interaction.

SurrogateAgent        Stage 7 (optional, after 50 experiments). Trains
                      or retrains a Gaussian Process surrogate model.
                      Tools: read_db_experiments, fit_gp_model,
                      persist_model.

SwarmSyncAgent        Stage 9. Sends heartbeat, pushes experiment records
                      to coordinator, pulls global_best_M from Redis.
                      Tools: http_patch_heartbeat, http_post_experiment,
                      redis_getset_global_best.

GEP Sub-Agents (run during Stage 8 evolution, blocking)
────────────────────────────────────────────────────────
GEPEvaluatorAgent     Computes fitness for each candidate genome from
                      ExperimentRegistry (history only, no new CFD).

GEPMutatorAgent       Applies single-gene mutation to candidate genomes.

GEPCrossoverAgent     Applies uniform crossover between genome pairs.

GEPSelectorAgent      Selects next generation (elitism + crossover +
                      mutation + random; population = 16).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MESSAGE PROTOCOL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Base LoopMessage schema (TypeScript):

  interface LoopMessage {
    message_id:    string;       // UUID v4
    run_id:        string;       // UUID v4
    experiment_n:  number;       // monotonic integer
    node_id:       string;       // UUID v4
    timestamp:     string;       // ISO 8601 UTC
    message_type:  MessageType;  // see GLOBAL-MSG-001
    stage:         Stage;        // 0–9
    payload:       Record<string, unknown>;
    error?:        { code: string; message: string; stack?: string };
  }

Full MessageType enum and Stage enum are defined in GLOBAL-MSG-001.

BullMQ queue layout:

  aeroloop-{run_id}-core          Serial; one job per stage per experiment.
  aeroloop-{run_id}-background    Parallel; KG, wiki, swarm sync.
  aeroloop-{run_id}-gep           Serial; GEP evolution (blocks core).
  aeroloop-swarm-heartbeat        Repeated; 30 s interval per node.

Core queue job options:
  attempts:          1           (never retry; always revert and continue)
  timeout:           (KILL_TIMER_S + 60) × 1000 ms
  removeOnComplete:  200
  removeOnFail:      500


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: program.md
PROGRAMME OVERVIEW
═══════════════════════════════════════════════════════════════════════

AeroLoop is an autonomous aerodynamic optimisation system. It searches
the wing geometry design space by running a tight, repeating loop:
propose one geometry change → mesh → CFD solve → evaluate → keep or
revert. Over many experiments it compounds learning via a knowledge
graph, a self-compiled wiki, and a GEP-based strategy evolver.

Design philosophy
─────────────────
• One experiment = one geometry change. Compound mutations are
  forbidden in normal mode.
• Every experiment ends in a reproducible, auditable state (KEPT
  or REVERTED). The git log is a monotonically improving record.
• Learning is explicit: the mutation agent reads KG summaries and
  wiki articles before proposing, so the system teaches itself.
• The strategy itself evolves: GEP adapts how the system searches,
  not just what it searches.
• Unattended safety: kill timers, dirty-tree halts, consecutive-
  failure pauses, and provenance laws keep the system safe overnight.

Technology stack
────────────────
  Runtime:        Bun
  Language:       TypeScript (strict mode, no any)
  API framework:  Hono (apps/api)
  CLI framework:  Bun CLI (apps/cli)
  ORM:            Drizzle ORM
  Queue:          BullMQ (backed by Redis)
  Database:       PostgreSQL 15
  Object storage: S3-compatible (MinIO in dev, AWS S3 in prod)
  Mesher:         GMSH 4.11+ (via Python subprocess)
  CFD solver:     SU2 7.5+  (via mpirun subprocess)
  Monorepo:       Turborepo + pnpm workspaces
  Testing:        Vitest
  Containers:     Docker / Kubernetes

Implementation roadmap
──────────────────────
Phase 0 — Foundation (Week 1–2)
  Goal: working monorepo skeleton; all shared types; DB schema; git
  integrity primitives.
  Deliverables:
  [ ] pnpm workspace + Turborepo setup
  [ ] packages/types — all Zod schemas compiled and exported
  [ ] packages/db — Drizzle schema, migrations, append-only trigger
  [ ] packages/redis — client, key helpers, SETNX claim helpers
  [ ] packages/git-ops — clean-check, keep (commit), revert (checkout)
      (100% unit tested)
  [ ] packages/geometry — parameter registry, bounds checker,
      wing.geo parser (95% unit tested)
  [ ] hooks/pre-commit + hooks/commit-msg — installed and tested
  [ ] infra/docker-compose.yml — postgres, redis, minio running
  [ ] CI pipeline — typecheck + lint + unit tests passing
  Milestone gate: packages/git-ops integration test passes (clean-check
  + keep + revert cycle on real git repo with wing.geo).

Phase 1 — Core Pipeline (Week 3–5)
  Goal: single experiment runs end-to-end (Stages 0–5), real GMSH,
  mocked SU2.
  Deliverables:
  [ ] packages/meshing — auto_mesh.py wrapper, quality checker, S3
      upload with SHA256 (medium level initially)
  [ ] packages/solver — SU2Runner interface, MockSU2Runner, dual
      kill-timer, convergence checker
  [ ] packages/extraction — polar extractor, wave drag, metric
      computer (95% unit tested with reference aerodynamic data)
  [ ] apps/loop — Stage 0–5 orchestrator, BullMQ core queue wired
  [ ] apps/api — /runs and /experiments endpoints (POST + GET)
  [ ] Integration tests IT-001 through IT-004 passing
  Milestone gate: 50-experiment autonomous run completes overnight
  with ≥1 KEEP, zero git integrity violations, full append-only
  experiment history in DB.

Phase 2 — Learning Layer (Week 6–8)
  Goal: knowledge graph, wiki, and informed mutations working.
  Deliverables:
  [ ] packages/knowledge-graph — updater, impact detector, interaction
      discovery (Mann–Whitney U, p < 0.05), dead/hot zone tracking
  [ ] packages/wiki — sensitivity + batch + interaction article
      compilers (enforce specific-numbers-and-experiment-refs rule)
  [ ] apps/api — /knowledge-graph and /wiki endpoints
  [ ] apps/loop — Stages 6–7 wired into background queue
  [ ] GeometryMutationAgent reads KG summary + wiki before proposing
  [ ] Anti-repetition check (last 10 experiments) implemented
  Milestone gate: 200-experiment run shows decreasing mutation attempts
  in declared dead zones (verifiable from experiment log).

Phase 3 — Adaptive Strategy (Week 9–10)
  Goal: GEP evolver working; genome evolves after 500 experiments.
  Deliverables:
  [ ] packages/surrogate — GP regression, train every 50 experiments,
      advisory input to GeometryMutationAgent only
  [ ] packages/gep — genome schema, fitness (history-only), mutation,
      crossover, selection (population = 16, elites = 2)
  [ ] apps/loop — Stage 8 wired; loop pauses at n % 500 === 0
  [ ] GEP integration test IT-005 passing
  [ ] Topology macro-mutation gated by topology_probability
  Milestone gate: IT-005 passes; genome committed with correct message
  format; loop resumes without manual intervention.

Phase 4 — Swarm (Week 11–13)
  Goal: multi-node swarm with non-collision and shared learning.
  Deliverables:
  [ ] apps/api — /swarm endpoints + SSE stream
  [ ] apps/loop — Stage 9 swarm guard; region claim/release
  [ ] apps/dashboard — React + Vite dashboard consuming SSE
  [ ] apps/cli — all operator commands
  [ ] OFFLINE_MODE + reconnect
  [ ] Integration test IT-006 passing (4-node non-collision)
  [ ] Kubernetes manifests (loop StatefulSet, api Deployment, HPA)
  Milestone gate: 4-node swarm completes 500 experiments with zero git
  conflicts and all records reconciled in shared DB.

Phase 5 — Production Hardening (Week 14–16)
  Goal: production-ready; real SU2 on real geometry.
  Deliverables:
  [ ] Replace MockSU2Runner with real SU2 in all integration tests
      (NACA 0012 — fast solve, known polars)
  [ ] Full mesh level support (coarse / medium / fine)
  [ ] Alpha sweep for buffet onset fully wired
  [ ] Terraform / Helm chart for cloud deployment
  [ ] Release pipeline; v1.0.0 tagged
  [ ] 1000-experiment unattended overnight test on real wing.geo
  Milestone gate: 1000-experiment autonomous run shows monotonically
  non-decreasing best_M; < 5% experiment loss rate (timeout + mesh
  fail combined).


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/architecture.md
GLOBAL-ARCH-001 — GLOBAL ARCHITECTURE
═══════════════════════════════════════════════════════════════════════

System overview
───────────────
AeroLoop consists of one or more autonomous loop nodes sharing a
central data layer (PostgreSQL, Redis, S3). Each node runs an
independent optimisation loop. Shared learning flows through the
knowledge graph and shared ExperimentRegistry. Swarm coordination
prevents duplicate work via Redis region claims.

Component map
─────────────
  ┌─────────────────────────────────────────────────────────┐
  │  Loop Node (1..N)                                       │
  │  ┌────────────┐  ┌──────────┐  ┌─────────┐             │
  │  │ apps/loop  │  │ GMSH     │  │ SU2     │             │
  │  │ (Bun)      │→ │ mesher   │→ │ solver  │             │
  │  └────────────┘  └──────────┘  └─────────┘             │
  │        │                                                │
  │  ┌─────▼──────┐  ┌──────────────────────────────┐      │
  │  │ BullMQ     │  │ geometry/wing.geo (git)       │      │
  │  │ (Redis)    │  │ (ONE mutable file per node)   │      │
  │  └────────────┘  └──────────────────────────────┘      │
  └─────────────────────────────────────────────────────────┘
        │                    │                    │
  ┌─────▼────┐       ┌───────▼──────┐    ┌───────▼──────┐
  │PostgreSQL│       │ Redis        │    │ S3           │
  │(runs,    │       │(claims,locks,│    │(meshes,      │
  │ exps, kg,│       │ global_best, │    │ results)     │
  │ wiki)    │       │ heartbeats)  │    │              │
  └──────────┘       └──────────────┘    └──────────────┘
        │
  ┌─────▼─────────────────────────────────────────────┐
  │  apps/api (Hono, Bun)                             │
  │  REST + SSE  →  apps/dashboard (React/Vite)       │
  │              →  apps/cli (Bun CLI)                │
  └───────────────────────────────────────────────────┘

Experiment data flow (per node, per experiment)
───────────────────────────────────────────────
  wing.geo (read)
    → GeometryMutationAgent (proposes 1 mutation, reads KG + wiki)
    → BoundsCheckAgent (validates against registry hard bounds)
    → wing.geo (write — ONLY if bounds pass)
    → MeshAgent (GMSH auto_mesh.py → SU2 format → S3 + SHA256)
    → MeshQualityAgent (quality gate — all thresholds must pass)
    → SolverAgent (SU2 + dual kill timer → surface_*.csv, history_*.csv → S3 + SHA256)
    → ExtractionAgent (CL, CD, CMy, L/D, Cd_wave, buffet onset)
    → MetricAgent (composite M)
    → KeepRevertAgent (RULE-008: M_new > M_best? → KEEP commit or REVERT checkout)
    → [background] KnowledgeGraphAgent, WikiCompilerAgent, SwarmSyncAgent
    → [every 500] GEPOrchestrator (blocking)
    → repeat

Throughput targets (single node, medium mesh, 8-minute kill timer)
────────────────────────────────────────────────────────────────────
  Typical solve:        4–7 minutes
  Full experiment:      6–10 minutes (including mesh + extract + overhead)
  Target rate:          6–10 experiments/hour
  Overnight (10 h):     60–100 experiments
  With 4-node swarm:    240–400 experiments/overnight

These are targets, not guarantees. Actual throughput depends on mesh
robustness, solver stability, cluster contention, and strategy aggressiveness.


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/data-model.md
GLOBAL-DM-001 — COMPLETE DATABASE SCHEMA
═══════════════════════════════════════════════════════════════════════

All tables use PostgreSQL 15. Drizzle ORM is the query builder.
All UUIDs use gen_random_uuid(). All timestamps are TIMESTAMPTZ UTC.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: experiment_runs
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE experiment_runs (
  run_id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_label             TEXT NOT NULL,
  status                TEXT NOT NULL CHECK (status IN (
                          'INITIALISING','RUNNING','PAUSED',
                          'COMPLETED','FAILED')),
  node_id               UUID NOT NULL,

  -- Locked config (immutable after INITIALISING → RUNNING transition)
  metric_weights        JSONB NOT NULL,
  operating_conditions  JSONB NOT NULL,
  solver_config         JSONB NOT NULL,
  mesh_config           JSONB NOT NULL,
  metric_version        TEXT NOT NULL DEFAULT '1.0',

  -- Progress counters
  total_experiments     INTEGER NOT NULL DEFAULT 0,
  kept_count            INTEGER NOT NULL DEFAULT 0,
  reverted_count        INTEGER NOT NULL DEFAULT 0,
  timeout_count         INTEGER NOT NULL DEFAULT 0,
  mesh_fail_count       INTEGER NOT NULL DEFAULT 0,
  consecutive_fail_count INTEGER NOT NULL DEFAULT 0,

  -- Best-so-far (updated on every KEEP)
  best_M                DOUBLE PRECISION,
  best_experiment_n     INTEGER,
  best_geometry_sha256  TEXT,

  -- GEP state
  current_genome        JSONB,
  gep_generation        INTEGER NOT NULL DEFAULT 0,
  gep_in_progress       BOOLEAN NOT NULL DEFAULT FALSE,

  -- Swarm
  swarm_enabled         BOOLEAN NOT NULL DEFAULT FALSE,

  -- Baseline
  baseline_experiment_id UUID,

  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Valid status transitions enforced in application layer:
--   INITIALISING → RUNNING
--   RUNNING → PAUSED | COMPLETED | FAILED
--   PAUSED  → RUNNING | COMPLETED

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: experiments  (APPEND-ONLY — enforced by trigger below)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE experiments (
  experiment_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id                 UUID NOT NULL REFERENCES experiment_runs(run_id),
  experiment_n           INTEGER NOT NULL,
  node_id                UUID NOT NULL,

  -- Mutation
  mutation_parameter_id  TEXT NOT NULL,
  mutation_value_before  DOUBLE PRECISION NOT NULL,
  mutation_value_after   DOUBLE PRECISION NOT NULL,
  mutation_delta         DOUBLE PRECISION NOT NULL,
  mutation_strategy      TEXT NOT NULL,
  topology_mutation      BOOLEAN NOT NULL DEFAULT FALSE,

  -- Geometry provenance
  geometry_sha256_before TEXT NOT NULL,
  geometry_sha256_after  TEXT,         -- null if experiment never wrote wing.geo

  -- Stage 2: Mesh
  bounds_valid           BOOLEAN,
  mesh_quality_ok        BOOLEAN,
  mesh_cell_count        INTEGER,
  mesh_s3_key            TEXT,
  mesh_sha256            TEXT,

  -- Stage 3: Solve
  solve_converged        BOOLEAN,
  solve_wall_time_s      DOUBLE PRECISION,
  solve_timeout          BOOLEAN NOT NULL DEFAULT FALSE,
  results_s3_key         TEXT,
  results_sha256         TEXT,

  -- Stage 4: Aerodynamic coefficients
  cl                     DOUBLE PRECISION,
  cd                     DOUBLE PRECISION,
  cmy                    DOUBLE PRECISION,
  ld_ratio               DOUBLE PRECISION,
  cd_wave                DOUBLE PRECISION,
  buffet_onset_alpha     DOUBLE PRECISION,  -- null if alpha sweep not run

  -- Stage 4: Metric
  M                      DOUBLE PRECISION,
  M_best_at_time         DOUBLE PRECISION,
  delta_M                DOUBLE PRECISION,

  -- Stage 5: Decision
  status                 TEXT NOT NULL CHECK (status IN (
                           'PENDING','BOUNDS_FAIL','MESH_FAIL',
                           'SOLVE_TIMEOUT','SOLVE_FAIL','EXTRACT_FAIL',
                           'REVERTED','KEPT','ABORTED')),
  git_commit_hash        TEXT,           -- null unless status = KEPT

  -- GEP context
  genome_generation      INTEGER,

  -- Timing
  stage_timings          JSONB,
    -- { bounds_ms, mesh_ms, solve_ms, extract_ms, decision_ms }
  total_wall_time_s      DOUBLE PRECISION,

  created_at             TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (run_id, experiment_n)  -- monotonicity + uniqueness
);

-- Append-only enforcement
CREATE OR REPLACE FUNCTION prevent_experiment_update()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'UPDATE' THEN
    RAISE EXCEPTION
      'experiments table is append-only; UPDATE is forbidden';
  END IF;
  RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER experiments_append_only
  BEFORE UPDATE OR DELETE ON experiments
  FOR EACH ROW EXECUTE FUNCTION prevent_experiment_update();

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: genomes  (APPEND-ONLY — one row per evolved generation)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE genomes (
  genome_id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id                UUID NOT NULL REFERENCES experiment_runs(run_id),
  generation            INTEGER NOT NULL,
  evolved_at_experiment_n INTEGER NOT NULL,
  strategy              TEXT NOT NULL,
  parameter_weights     JSONB NOT NULL,
  step_size_multiplier  DOUBLE PRECISION NOT NULL,
  patience              INTEGER NOT NULL,
  topology_probability  DOUBLE PRECISION NOT NULL,
  fitness_before        DOUBLE PRECISION,
  fitness_after         DOUBLE PRECISION NOT NULL,
  population_snapshot   JSONB NOT NULL,  -- all 16 candidates
  git_commit_hash       TEXT NOT NULL,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (run_id, generation)
);

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: knowledge_graph_entries  (raw per-experiment observation)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE knowledge_graph_entries (
  entry_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id            UUID REFERENCES experiment_runs(run_id),
  parameter_id      TEXT NOT NULL,
  experiment_n      INTEGER NOT NULL,
  node_id           UUID NOT NULL,
  delta_value       DOUBLE PRECISION NOT NULL,
  delta_M           DOUBLE PRECISION NOT NULL,
  outcome           TEXT NOT NULL CHECK (outcome IN (
                      'KEPT','REVERTED','TIMEOUT','MESH_FAIL','BOUNDS_FAIL')),
  mesh_quality_ok   BOOLEAN,
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX kg_param_run ON knowledge_graph_entries(parameter_id, run_id);

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: knowledge_graph_sensitivities  (computed summary per parameter)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE knowledge_graph_sensitivities (
  sensitivity_id    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id            UUID REFERENCES experiment_runs(run_id),
  parameter_id      TEXT NOT NULL,
  sensitivity_score DOUBLE PRECISION NOT NULL,
  direction         TEXT CHECK (direction IN ('positive','negative','neutral')),
  confidence        DOUBLE PRECISION NOT NULL,
  sample_count      INTEGER NOT NULL,
  dead_zone         BOOLEAN NOT NULL DEFAULT FALSE,
  hot_zone          BOOLEAN NOT NULL DEFAULT FALSE,
  computed_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (run_id, parameter_id)  -- upserted after each experiment
);

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: knowledge_graph_interactions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE knowledge_graph_interactions (
  interaction_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id               UUID REFERENCES experiment_runs(run_id),
  param_a              TEXT NOT NULL,
  param_b              TEXT NOT NULL,
  interaction_strength DOUBLE PRECISION NOT NULL,
  p_value              DOUBLE PRECISION NOT NULL,
  direction            TEXT CHECK (direction IN (
                         'synergistic','antagonistic','neutral')),
  sample_count         INTEGER NOT NULL,
  discovered_at_exp    INTEGER NOT NULL,
  computed_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (run_id, param_a, param_b)
);

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: wiki_articles
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE wiki_articles (
  article_id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id                  UUID REFERENCES experiment_runs(run_id),
  title                   TEXT NOT NULL,
  type                    TEXT NOT NULL CHECK (type IN (
                            'SENSITIVITY','BATCH_SUMMARY',
                            'INTERACTION','TOPOLOGY')),
  experiment_range_start  INTEGER,
  experiment_range_end    INTEGER,
  referenced_parameters   TEXT[],
  referenced_experiments  INTEGER[],
  file_path               TEXT NOT NULL,
  body_markdown           TEXT NOT NULL,
  created_at              TIMESTAMPTZ NOT NULL DEFAULT now()
);

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TABLE: swarm_nodes
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE swarm_nodes (
  node_id          UUID PRIMARY KEY,
  run_id           UUID REFERENCES experiment_runs(run_id),
  hostname         TEXT NOT NULL,
  status           TEXT NOT NULL CHECK (status IN (
                     'ACTIVE','IDLE','SOLVING','GEP','STALE','OFFLINE')),
  current_region   TEXT,
  local_best_M     DOUBLE PRECISION,
  experiment_n     INTEGER,
  core_count       INTEGER,
  memory_gb        DOUBLE PRECISION,
  gpu_available    BOOLEAN DEFAULT FALSE,
  last_heartbeat   TIMESTAMPTZ NOT NULL DEFAULT now(),
  registered_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  deregistered_at  TIMESTAMPTZ
);

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DATABASE INDEXES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE INDEX exp_run_n      ON experiments(run_id, experiment_n DESC);
CREATE INDEX exp_run_status ON experiments(run_id, status);
CREATE INDEX exp_node       ON experiments(node_id);
CREATE INDEX exp_kept       ON experiments(run_id) WHERE status = 'KEPT';
CREATE INDEX wiki_run_type  ON wiki_articles(run_id, type);
CREATE INDEX swarm_run      ON swarm_nodes(run_id, status);
CREATE INDEX kg_run_param   ON knowledge_graph_sensitivities(run_id, parameter_id);


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/geometry.md
GLOBAL-GEOM-001 — GEOMETRY PARAMETER REGISTRY
═══════════════════════════════════════════════════════════════════════

This registry is the single source of truth for:
  (a) the complete list of optimisable parameters
  (b) their hard physical bounds (inviolable — see RULE-004)
  (c) default values
  (d) step sizes per search strategy

Format: parameter_id | min | max | default | step_fine | step_medium | step_global

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY: PLANFORM (core)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
span_m               |  20.0 |  80.0 |  35.0 |  0.10 |  0.50 |  2.00
aspect_ratio         |   5.0 |  16.0 |   9.0 |  0.05 |  0.20 |  1.00
taper_ratio          |   0.2 |   0.7 |   0.4 |  0.01 |  0.03 |  0.10
le_sweep_inboard_deg |  10.0 |  45.0 |  27.5 |  0.20 |  0.80 |  3.00
le_sweep_outboard_deg|   5.0 |  40.0 |  22.0 |  0.20 |  0.80 |  3.00
te_sweep_deg         |  -5.0 |  20.0 |   5.0 |  0.20 |  0.80 |  3.00
dihedral_deg         |  -5.0 |  10.0 |   5.0 |  0.10 |  0.40 |  1.50
root_chord_m         |   4.0 |  12.0 |   7.5 |  0.05 |  0.20 |  0.80
tip_chord_m          |   1.0 |   5.0 |   2.5 |  0.05 |  0.10 |  0.40
mac_m                |   4.0 |  10.0 |   6.2 |  0.05 |  0.20 |  0.80

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY: TWIST DISTRIBUTION (span stations η = 0.0, 0.25, 0.5, 0.75, 1.0)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
twist_root_deg       |  -3.0 |   5.0 |   2.0 |  0.10 |  0.30 |  1.00
twist_25_deg         |  -3.0 |   5.0 |   1.5 |  0.10 |  0.30 |  1.00
twist_50_deg         |  -3.0 |   4.0 |   0.5 |  0.10 |  0.30 |  1.00
twist_75_deg         |  -4.0 |   2.0 |  -0.5 |  0.10 |  0.30 |  1.00
twist_tip_deg        |  -5.0 |   1.0 |  -1.5 |  0.10 |  0.30 |  1.00

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY: THICKNESS (t/c) DISTRIBUTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
tc_root              |  0.10 |  0.18 |  0.14 |  0.001 | 0.003 |  0.010
tc_25                |  0.09 |  0.16 |  0.13 |  0.001 | 0.003 |  0.010
tc_50                |  0.08 |  0.14 |  0.12 |  0.001 | 0.003 |  0.010
tc_75                |  0.07 |  0.13 |  0.11 |  0.001 | 0.003 |  0.010
tc_tip               |  0.06 |  0.12 |  0.10 |  0.001 | 0.003 |  0.010

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY: CAMBER DISTRIBUTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
camber_root          |  0.00 |  0.06 |  0.03 |  0.001 | 0.003 |  0.010
camber_25            |  0.00 |  0.06 |  0.03 |  0.001 | 0.003 |  0.010
camber_50            |  0.00 |  0.05 |  0.02 |  0.001 | 0.003 |  0.010
camber_75            |  0.00 |  0.04 |  0.02 |  0.001 | 0.003 |  0.010
camber_tip           |  0.00 |  0.04 |  0.01 |  0.001 | 0.003 |  0.010

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY: LEADING EDGE RADIUS DISTRIBUTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
le_radius_root       |  0.005|  0.030|  0.015|  0.001 | 0.002 |  0.005
le_radius_50         |  0.004|  0.025|  0.012|  0.001 | 0.002 |  0.005
le_radius_tip        |  0.003|  0.020|  0.010|  0.001 | 0.002 |  0.005

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY: WINGLET  (only active when topology_mutation = true OR
                    winglet already present in baseline)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
winglet_height_m     |  0.00 |  4.00 |  0.00 |  0.05 |  0.20 |  0.50
winglet_sweep_deg    |  10.0 |  60.0 |  35.0 |  0.50 |  2.00 |  5.00
winglet_cant_deg     |   0.0 |  90.0 |  15.0 |  1.00 |  3.00 | 10.00
winglet_toe_deg      |  -5.0 |   5.0 |   0.0 |  0.20 |  0.80 |  2.00
winglet_tc           |  0.06 |  0.12 |  0.09 |  0.001 | 0.003 |  0.010

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CROSS-PARAMETER CONSTRAINTS (enforced by BoundsCheckAgent)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. tip_chord_m < root_chord_m  (no reverse taper)
2. twist distribution must be monotonically non-increasing root→tip
   unless topology_mutation = true
3. t/c distribution must be monotonically non-increasing root→tip
4. aspect_ratio ≈ span_m² / (root_chord_m + tip_chord_m) × 2 / span_m
   (within 5% tolerance; enforced as a warning, not a hard block)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PRE-SEEDED KNOWN INTERACTIONS (seed the knowledge graph at run init)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
(span_m, aspect_ratio)           — synergistic (structural coupling)
(le_sweep_inboard_deg, tc_root)  — antagonistic (wave drag sensitivity)
(taper_ratio, twist_tip_deg)     — synergistic (induced drag coupling)
(camber_root, le_radius_root)    — synergistic (LE loading)
(winglet_height_m, span_m)       — synergistic (effective span)

These are confidence=0.3 seed entries; real data overwrites them.
Part 1 is complete. It covers:

✅ CLAUDE.md — all 12 constitutional rules, fully consistent (RULE-003 now says git checkout, RULE-060 and RULE-061 are the single canonical commit formats, no duplicate/conflicting formats)
✅ AGENTS.md — full agent registry with corrected tool name (git_checkout_restore, not git_revert); BullMQ queue layout; GEP population=16 stated in the agent descriptions to seed consistency
✅ program.md — stack declared as Bun + Hono (no FastAPI/Hono slash ambiguity); roadmap with Phase 0–5 and milestone gates; GEP population=16 in Phase 3 deliverables
✅ GLOBAL-ARCH-001 — data flow, component map, throughput targets
✅ GLOBAL-DM-001 — complete SQL schema including the missing genomes table, all 6 tables, append-only trigger, indexes
✅ GLOBAL-GEOM-001 — full parameter registry with all categories + cross-parameter constraints + pre-seeded interactions




Part 2 of 3 — covering all global specification files and Stages 0 through 5.

Markdown

═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/api-contracts.md
GLOBAL-API-001 — API CONTRACT SPECIFICATION
═══════════════════════════════════════════════════════════════════════

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONVENTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Framework:       Hono (Bun runtime) — apps/api
Base URL (dev):  http://localhost:4000/api/v1
Base URL (prod): https://aeroloop.internal/api/v1

All requests/responses: Content-Type: application/json
All timestamps:         ISO 8601 UTC string
                        e.g. "2025-01-15T14:32:00.000Z"
All UUIDs:              UUID v4 string

Authentication
  Header:  Authorization: Bearer {jwt}
  Exempt:  GET /health, GET /swarm/stream (dev mode only)
  On fail: 401 { "type": "...", "status": 401, "title": "Unauthorised" }

Error format (RFC 7807 Problem Details — all 4xx and 5xx):
  {
    "type":     "https://aeroloop.internal/errors/{code}",
    "title":    "Human readable title",
    "status":   422,
    "detail":   "Specific detail string",
    "instance": "/api/v1/experiments/run-id/144"
  }

Pagination: cursor-based
  Query:    ?after={cursor}&limit={n}   (max limit: 200, default: 50)
  Response: { "data": [...], "next_cursor": {value} | null, "total": n }

Rate limiting:
  1000 requests/min per node_id (identified by JWT sub claim)
  429 Too Many Requests on breach; Retry-After header included

Input validation: all request bodies validated with Zod schemas
  on the server; invalid bodies return 422 with field-level errors

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API-001: HEALTH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GET /health
  No auth required.

  Response 200:
    {
      "status":    "ok",
      "db":        "ok",
      "redis":     "ok",
      "s3":        "ok",
      "version":   "1.0.0",
      "uptime_s":  3600
    }

  Response 503 (any dependency degraded or down):
    Same shape; failing service value is "degraded" or "down".
    "status" is "degraded" if any service is not "ok".

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API-002: EXPERIMENT RUNS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

POST /runs
  Create and lock a new ExperimentRun. Called by loop orchestrator
  at Stage 0 initialisation. Config is immutable after creation.

  Request body:
    {
      "node_id":    "uuid-v4",
      "run_label":  "naca0012-baseline-run-1",

      "metric_weights": {
        "w_ld":     0.5,
        "w_buffet": 0.3,
        "w_wave":   0.2
      },

      "operating_conditions": {
        "mach":        0.78,
        "reynolds":    4.5e7,
        "alpha_cruise": 2.5,
        "altitude_m":  11000
      },

      "solver_config": {
        "solver":                "SU2",
        "version":               "7.5.1",
        "cfl_number":            10.0,
        "max_iter":              3000,
        "convergence_cauchy_eps": 1e-6,
        "mpi_ranks":             8,
        "wall_time_limit_s":     480,
        "turbulence_model":      "SA"
      },

      "mesh_config": {
        "mesher":        "GMSH",
        "version":       "4.11.1",
        "default_level": "medium",
        "y_plus_target": 1.0
      },

      "swarm_enabled": false,
      "gep_enabled":   true
    }

  Validation rules (all enforced server-side; 422 on breach):
    - metric_weights values must all be >= 0
    - metric_weights must sum to 1.0 ± 0.001
    - w_ld must be >= 0.3
    - mach in (0.0, 1.5]
    - reynolds in [1e5, 1e9]
    - wall_time_limit_s in [60, 3600]
    - mpi_ranks in [1, 256]

  Response 201:
    {
      "run_id":                 "uuid-v4",
      "created_at":             "2025-01-15T14:00:00.000Z",
      "status":                 "INITIALISING",
      "baseline_experiment_id": null
    }

  Errors:
    409  node_id already has an active run (status RUNNING or PAUSED)
    422  validation failure (field-level errors in detail)

───────────────────────────────────────────────────────────────────────

GET /runs/{run_id}
  Returns full ExperimentRun record with current progress stats.

  Response 200:
    {
      "run_id":               "uuid-v4",
      "run_label":            "naca0012-baseline-run-1",
      "status":               "RUNNING",
      "metric_weights":       { "w_ld": 0.5, "w_buffet": 0.3, "w_wave": 0.2 },
      "operating_conditions": { ... },
      "solver_config":        { ... },
      "mesh_config":          { ... },
      "metric_version":       "1.0",
      "total_experiments":    143,
      "kept_count":           41,
      "reverted_count":       99,
      "timeout_count":        2,
      "mesh_fail_count":      1,
      "consecutive_fail_count": 0,
      "best_M":               0.8821,
      "best_experiment_n":    138,
      "best_geometry_sha256": "sha256-hex-string",
      "current_genome":       { ...genome fields... },
      "gep_generation":       0,
      "swarm_enabled":        false,
      "created_at":           "...",
      "updated_at":           "..."
    }

  Errors: 404 run_id not found

───────────────────────────────────────────────────────────────────────

PATCH /runs/{run_id}
  Update run status. Used by orchestrator and operator commands.

  Request body:
    { "status": "PAUSED", "reason": "operator_halt" }

  Valid transitions (server enforces; 422 on invalid):
    INITIALISING → RUNNING
    RUNNING      → PAUSED | COMPLETED | FAILED
    PAUSED       → RUNNING | COMPLETED

  Response 200: updated ExperimentRun record (same shape as GET)
  Errors: 404, 422 (invalid transition)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API-003: EXPERIMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

POST /experiments
  Append a new experiment record. APPEND-ONLY; no PUT or DELETE.
  Called by KeepRevertAgent at the end of Stage 5.

  Request body (all fields from GLOBAL-DM-001 experiments table):
    {
      "run_id":                  "uuid-v4",
      "experiment_n":            144,
      "node_id":                 "uuid-v4",
      "mutation_parameter_id":   "taper_ratio",
      "mutation_value_before":   0.41,
      "mutation_value_after":    0.38,
      "mutation_delta":          -0.03,
      "mutation_strategy":       "fine_search",
      "topology_mutation":       false,
      "geometry_sha256_before":  "abc123...",
      "geometry_sha256_after":   "def456...",
      "bounds_valid":            true,
      "mesh_quality_ok":         true,
      "mesh_cell_count":         2140000,
      "mesh_s3_key":             "runs/{run_id}/exp-144/mesh.su2",
      "mesh_sha256":             "ghi789...",
      "solve_converged":         true,
      "solve_wall_time_s":       312.4,
      "solve_timeout":           false,
      "results_s3_key":          "runs/{run_id}/exp-144/results/",
      "results_sha256":          "jkl012...",
      "cl":                      0.512,
      "cd":                      0.0271,
      "cmy":                     -0.014,
      "ld_ratio":                18.89,
      "cd_wave":                 0.0031,
      "buffet_onset_alpha":      4.2,
      "M":                       0.8821,
      "M_best_at_time":          0.8789,
      "delta_M":                 0.0032,
      "status":                  "KEPT",
      "git_commit_hash":         "a1b2c3d...",
      "genome_generation":       0,
      "stage_timings": {
        "bounds_ms":   12,
        "mesh_ms":     84200,
        "solve_ms":    312400,
        "extract_ms":  340,
        "decision_ms": 890
      },
      "total_wall_time_s":       397.8
    }

  Required fields (minimum for non-KEPT experiments):
    run_id, experiment_n, node_id, mutation_parameter_id,
    mutation_value_before, mutation_value_after, mutation_delta,
    mutation_strategy, topology_mutation, geometry_sha256_before, status

  Response 201:
    { "experiment_id": "uuid-v4", "experiment_n": 144 }

  Errors:
    409  (run_id, experiment_n) already exists
    422  experiment_n is not exactly max(experiment_n) + 1 for this
         run_id (monotonicity enforced at DB constraint level)
    422  status not in allowed enum values
    404  run_id not found

───────────────────────────────────────────────────────────────────────

GET /experiments/{run_id}
  List experiments for a run with pagination and filtering.

  Query params:
    ?status=KEPT          filter by status (any valid status value)
    ?after=143&limit=50   cursor pagination (experiment_n as cursor)
    ?node_id={uuid}       filter by node (swarm use)
    ?sort=desc            reverse chronological (default: asc)

  Response 200:
    {
      "data":        [ { ...ExperimentRecord... } ],
      "next_cursor": 195,
      "total":       144
    }

───────────────────────────────────────────────────────────────────────

GET /experiments/{run_id}/{experiment_n}
  Single experiment record.

  Response 200: full ExperimentRecord (all fields, same shape as POST body)
  Errors: 404

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API-004: KNOWLEDGE GRAPH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GET /knowledge-graph/summary
  Return parameter sensitivity and interaction summary.
  Called by GeometryMutationAgent before every mutation proposal.

  Query params:
    ?run_id={uuid}                      scope to a run (required)
    ?region=REGION_PLANFORM_CORE        filter by swarm region (optional)
    ?parameter_ids=span,taper_ratio     filter to specific params (optional)
    ?node_id={uuid}                     logged for audit (optional)

  Timeout contract: response MUST arrive within 3 s. On timeout,
  the calling agent uses its locally cached summary (max staleness:
  10 minutes). The API server must set a server-side 2.8 s query
  timeout to leave headroom for network.

  Response 200:
    {
      "generated_at": "2025-01-15T15:00:00.000Z",
      "parameters": [
        {
          "parameter_id":      "span_m",
          "sensitivity_score": 0.72,
          "direction":         "positive",
          "confidence":        0.85,
          "sample_count":      38,
          "dead_zone":         false,
          "hot_zone":          true
        }
      ],
      "interactions": [
        {
          "param_a":             "span_m",
          "param_b":             "taper_ratio",
          "interaction_strength": 0.61,
          "p_value":             0.031,
          "direction":           "synergistic"
        }
      ],
      "summary_markdown_path": ".aeroloop/knowledge_graph_summary.md"
    }

───────────────────────────────────────────────────────────────────────

POST /knowledge-graph/update
  Push a post-experiment knowledge graph observation.
  Called by KnowledgeGraphAgent after every experiment (Stage 7).

  Request body:
    {
      "run_id":          "uuid-v4",
      "experiment_n":    144,
      "node_id":         "uuid-v4",
      "parameter_id":    "taper_ratio",
      "delta_value":     -0.03,
      "delta_M":         0.0032,
      "outcome":         "KEPT",
      "mesh_quality_ok": true
    }

  Response 204: no body
  Errors: 422, 404

───────────────────────────────────────────────────────────────────────

POST /knowledge-graph/detect-impact
  Pre-mutation impact assessment.
  Called by GeometryMutationAgent BEFORE writing wing.geo.

  Request body:
    {
      "run_id":   "uuid-v4",
      "node_id":  "uuid-v4",
      "proposed_mutation": {
        "parameter_id":   "le_sweep_inboard_deg",
        "current_value":  27.5,
        "proposed_value": 29.1,
        "delta":          1.6
      }
    }

  Response 200:
    {
      "mesh_risk":   "LOW",
      "mesh_risk_reason": null,
      "known_interactions": [
        { "with_parameter": "tc_root", "strength": 0.61,
          "direction": "antagonistic" }
      ],
      "dead_zone":  false,
      "hot_zone":   false,
      "similar_experiments": [
        {
          "experiment_n": 98,
          "delta":        1.4,
          "outcome":      "KEPT",
          "delta_M":      0.002
        }
      ],
      "recommendation": "PROCEED",
      "confidence":     0.79,
      "warning":        null
    }

  mesh_risk values:     LOW | MEDIUM | HIGH
  recommendation values: PROCEED | PROCEED_WITH_CAUTION | SKIP

  Agent behaviour contract:
    - HIGH mesh_risk:        agent MUST log warning; MAY skip mutation
    - SKIP recommendation:  agent MAY skip; MUST log the skip decision
    - Skipping is never mandatory; logging is mandatory

  Errors: 422, 404

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API-005: WIKI
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GET /wiki/articles
  List wiki articles with pagination and filtering.

  Query params:
    ?run_id={uuid}
    ?type=SENSITIVITY|BATCH_SUMMARY|INTERACTION|TOPOLOGY
    ?parameter_id=span_m         articles referencing this parameter
    ?after={uuid}&limit=20

  Response 200:
    {
      "data": [
        {
          "article_id":    "uuid-v4",
          "title":         "Parameter Sensitivity: span_m",
          "type":          "SENSITIVITY",
          "experiment_range_start": 100,
          "experiment_range_end":   110,
          "referenced_parameters":  ["span_m", "taper_ratio"],
          "referenced_experiments": [100, 103, 107, 110],
          "created_at":    "...",
          "file_path":     ".aeroloop/wiki/sensitivity_span_exp100_110.md"
        }
      ],
      "next_cursor": "uuid-v4",
      "total": 14
    }

───────────────────────────────────────────────────────────────────────

GET /wiki/articles/{article_id}
  Returns article metadata and full markdown body.

  Response 200:
    {
      "article_id":              "uuid-v4",
      "title":                   "Parameter Sensitivity: span_m",
      "type":                    "SENSITIVITY",
      "experiment_range_start":  100,
      "experiment_range_end":    110,
      "referenced_parameters":   ["span_m", "taper_ratio"],
      "referenced_experiments":  [100, 103, 107, 110],
      "body_markdown":           "# Parameter Sensitivity: span_m\n...",
      "file_path":               ".aeroloop/wiki/...",
      "created_at":              "..."
    }

  Errors: 404

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API-006: SWARM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

POST /swarm/nodes
  Register a node at startup.

  Request body:
    {
      "node_id":     "uuid-v4",
      "run_id":      "uuid-v4",
      "hostname":    "worker-node-3",
      "core_count":  32,
      "memory_gb":   128.0,
      "gpu_available": false
    }

  Response 201:
    { "node_id": "uuid-v4", "registered_at": "..." }

  Errors:
    409 node_id already registered and not deregistered

───────────────────────────────────────────────────────────────────────

PATCH /swarm/nodes/{node_id}
  Heartbeat update. Called every 30 s by SwarmSyncAgent.

  Request body:
    {
      "status":       "SOLVING",
      "experiment_n": 144,
      "local_best_M": 0.8821,
      "current_region": "REGION_PLANFORM_CORE"
    }

  Response 200:
    {
      "global_best_M":  0.8831,
      "global_best_node_id": "uuid-v4"
    }

  Errors: 404 node_id not found or already deregistered

───────────────────────────────────────────────────────────────────────

DELETE /swarm/nodes/{node_id}
  Deregister node on clean shutdown. Releases all region claims.
  Response 204. Errors: 404.

───────────────────────────────────────────────────────────────────────

GET /swarm/nodes
  List all nodes for a run.

  Query params: ?run_id={uuid}&status=ACTIVE|IDLE|SOLVING|GEP|STALE

  Response 200:
    {
      "data": [
        {
          "node_id":        "uuid-v4",
          "hostname":       "worker-node-3",
          "status":         "SOLVING",
          "current_region": "REGION_TWIST",
          "local_best_M":   0.871,
          "experiment_n":   143,
          "last_heartbeat": "2025-01-15T15:01:00.000Z"
        }
      ]
    }

───────────────────────────────────────────────────────────────────────

GET /swarm/regions
  List all regions and their current claim status for a run.

  Query params: ?run_id={uuid} (required)

  Response 200:
    {
      "regions": [
        {
          "region_name":    "REGION_PLANFORM_CORE",
          "owner_node_id":  "uuid-v4",
          "claimed_at":     "2025-01-15T14:55:00.000Z",
          "ttl_remaining_s": 341
        },
        {
          "region_name":    "REGION_TWIST",
          "owner_node_id":  null,
          "claimed_at":     null,
          "ttl_remaining_s": null
        }
      ]
    }

───────────────────────────────────────────────────────────────────────

GET /swarm/stream
  Server-Sent Events stream of real-time swarm events.
  Auth: Bearer JWT in query param ?token={jwt} (SSE does not support
  custom headers in browser EventSource).
  No auth required in NODE_ENV=development.

  Event format (text/event-stream):
    event: {MessageType}
    data: {"run_id":"...","experiment_n":144,"node_id":"...",
           "delta_M":0.004,"M_new":0.882,"timestamp":"..."}

  Emitted event types:
    NODE_HEARTBEAT, EXPERIMENT_KEPT, EXPERIMENT_REVERTED,
    REGION_CLAIMED, REGION_RELEASED, GLOBAL_BEST_UPDATED,
    NODE_STALE, GEP_EVOLUTION_START, GEP_EVOLUTION_COMPLETE,
    LOOP_CONSECUTIVE_FAIL, LOOP_HALT_DIRTY_TREE

  Latency contract: events emitted within 500 ms of occurrence.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API-007: OPERATOR COMMANDS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

POST /operator/pause
  Pause the loop on one or all nodes gracefully. The loop pauses
  AFTER the current experiment completes (not mid-stage).

  Request body:
    { "run_id": "uuid-v4", "node_id": "uuid-v4|ALL", "reason": "manual_review" }

  Response 202: accepted (asynchronous; node pauses on next iteration)
  Errors: 404

───────────────────────────────────────────────────────────────────────

POST /operator/resume
  Resume one or all paused nodes.

  Request body:
    { "run_id": "uuid-v4", "node_id": "uuid-v4|ALL" }

  Response 202. Errors: 404, 422 (node not in PAUSED status)

───────────────────────────────────────────────────────────────────────

POST /operator/abort-experiment
  Abort the current experiment on a node immediately. Kill timer
  fires; geometry is reverted via git checkout; experiment is
  recorded as ABORTED; loop continues to next experiment.

  Request body:
    { "run_id": "uuid-v4", "node_id": "uuid-v4" }

  Response 202. Errors: 404

───────────────────────────────────────────────────────────────────────

GET /operator/status
  Full system status: all runs, all nodes, global best,
  BullMQ queue depths, dependency health (DB/Redis/S3).

  Response 200:
    {
      "runs": [ { ...ExperimentRun summary... } ],
      "nodes": [ { ...SwarmNode summary... } ],
      "global_best_M": 0.8831,
      "queue_depths": {
        "core": 0, "background": 3, "gep": 0, "heartbeat": 4
      },
      "health": { "db": "ok", "redis": "ok", "s3": "ok" }
    }


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/messages.md
GLOBAL-MSG-001 — COMPLETE MESSAGE PROTOCOL
═══════════════════════════════════════════════════════════════════════

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STAGE ENUM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

enum Stage {
  INIT     = 0,
  MUTATION = 1,
  MESH     = 2,
  SOLVE    = 3,
  EXTRACT  = 4,
  DECISION = 5,
  WIKI     = 6,
  KG       = 7,
  GEP      = 8,
  SWARM    = 9,
}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MESSAGE TYPE ENUM (complete — all 9 stages)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

enum MessageType {

  // ── Loop lifecycle ───────────────────────────────────────────────
  LOOP_START                 = 'LOOP_START',
  LOOP_PAUSE                 = 'LOOP_PAUSE',
  LOOP_RESUME                = 'LOOP_RESUME',
  LOOP_HALT_DIRTY_TREE       = 'LOOP_HALT_DIRTY_TREE',
  LOOP_CONSECUTIVE_FAIL      = 'LOOP_CONSECUTIVE_FAIL',

  // ── Stage 0: Initialisation ──────────────────────────────────────
  RUN_INITIALISED            = 'RUN_INITIALISED',
  BASELINE_COMPLETE          = 'BASELINE_COMPLETE',
  BASELINE_FAILED            = 'BASELINE_FAILED',

  // ── Stage 1: Mutation ────────────────────────────────────────────
  MUTATION_PROPOSAL          = 'MUTATION_PROPOSAL',
  BOUNDS_VALID               = 'BOUNDS_VALID',
  BOUNDS_INVALID             = 'BOUNDS_INVALID',
  GEOMETRY_WRITTEN           = 'GEOMETRY_WRITTEN',
  MUTATION_SKIPPED_DEAD_ZONE = 'MUTATION_SKIPPED_DEAD_ZONE',
  MUTATION_SKIPPED_DUPLICATE = 'MUTATION_SKIPPED_DUPLICATE',

  // ── Stage 2: Mesh ────────────────────────────────────────────────
  MESH_STARTED               = 'MESH_STARTED',
  MESH_COMPLETE              = 'MESH_COMPLETE',
  MESH_QUALITY_PASS          = 'MESH_QUALITY_PASS',
  MESH_QUALITY_FAIL          = 'MESH_QUALITY_FAIL',
  MESH_UPLOADED              = 'MESH_UPLOADED',

  // ── Stage 3: Solve ───────────────────────────────────────────────
  SOLVE_STARTED              = 'SOLVE_STARTED',
  SOLVE_CONVERGED            = 'SOLVE_CONVERGED',
  SOLVE_UNCONVERGED          = 'SOLVE_UNCONVERGED',
  SOLVE_TIMEOUT              = 'SOLVE_TIMEOUT',
  SOLVE_FAILED               = 'SOLVE_FAILED',
  RESULTS_UPLOADED           = 'RESULTS_UPLOADED',

  // ── Stage 4: Extract + Metric ────────────────────────────────────
  EXTRACTION_COMPLETE        = 'EXTRACTION_COMPLETE',
  EXTRACTION_FAILED          = 'EXTRACTION_FAILED',
  METRICS_COMPUTED           = 'METRICS_COMPUTED',
  SANITY_WARNING             = 'SANITY_WARNING',

  // ── Stage 5: Keep / Revert ───────────────────────────────────────
  EXPERIMENT_KEPT            = 'EXPERIMENT_KEPT',
  EXPERIMENT_REVERTED        = 'EXPERIMENT_REVERTED',
  EXPERIMENT_ABORTED         = 'EXPERIMENT_ABORTED',
  GLOBAL_BEST_UPDATED        = 'GLOBAL_BEST_UPDATED',

  // ── Stage 6: Wiki ────────────────────────────────────────────────
  WIKI_COMPILE_STARTED       = 'WIKI_COMPILE_STARTED',
  WIKI_COMPILE_COMPLETE      = 'WIKI_COMPILE_COMPLETE',
  WIKI_ARTICLE_CREATED       = 'WIKI_ARTICLE_CREATED',

  // ── Stage 7: Knowledge Graph ─────────────────────────────────────
  KG_UPDATE_COMPLETE         = 'KG_UPDATE_COMPLETE',
  KG_INTERACTION_DISCOVERED  = 'KG_INTERACTION_DISCOVERED',
  KG_DEAD_ZONE_DECLARED      = 'KG_DEAD_ZONE_DECLARED',
  KG_HOT_ZONE_DECLARED       = 'KG_HOT_ZONE_DECLARED',
  KG_SURROGATE_TRAINED       = 'KG_SURROGATE_TRAINED',

  // ── Stage 8: GEP ─────────────────────────────────────────────────
  GEP_EVOLUTION_START        = 'GEP_EVOLUTION_START',
  GEP_EVALUATION_COMPLETE    = 'GEP_EVALUATION_COMPLETE',
  GEP_SELECTION_COMPLETE     = 'GEP_SELECTION_COMPLETE',
  GEP_EVOLUTION_COMPLETE     = 'GEP_EVOLUTION_COMPLETE',
  GEP_EVOLUTION_FAILED       = 'GEP_EVOLUTION_FAILED',
  TOPOLOGY_ENABLED           = 'TOPOLOGY_ENABLED',

  // ── Stage 9: Swarm ───────────────────────────────────────────────
  NODE_REGISTERED            = 'NODE_REGISTERED',
  NODE_HEARTBEAT             = 'NODE_HEARTBEAT',
  NODE_STALE                 = 'NODE_STALE',
  NODE_DEREGISTERED          = 'NODE_DEREGISTERED',
  REGION_CLAIMED             = 'REGION_CLAIMED',
  REGION_RELEASED            = 'REGION_RELEASED',
  REGION_CONFLICT_RESOLVED   = 'REGION_CONFLICT_RESOLVED',
  SWARM_OFFLINE_MODE         = 'SWARM_OFFLINE_MODE',
  SWARM_RECONNECTED          = 'SWARM_RECONNECTED',
}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BASE LOOPMESSAGE TYPE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

interface LoopMessage {
  message_id:   string;       // UUID v4
  run_id:       string;       // UUID v4
  experiment_n: number;       // monotonic integer
  node_id:      string;       // UUID v4
  timestamp:    string;       // ISO 8601 UTC
  message_type: MessageType;
  stage:        Stage;
  payload:      Record<string, unknown>;
  error?:       { code: string; message: string; stack?: string };
}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BULLMQ QUEUE LAYOUT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Queue                           Concurrency  Notes
────────────────────────────────────────────────────────────────────
aeroloop-{run_id}-core          1 (serial)   MUTATION→MESH→SOLVE→
                                             EXTRACT→DECISION per exp
aeroloop-{run_id}-background    3 (parallel) KG, wiki, swarm sync
aeroloop-{run_id}-gep           1 (serial)   blocks core during evolution
aeroloop-swarm-heartbeat        N (one per   30 s repeated job per node
                                 node)

Core queue job options:
  attempts:         1
    (never retry core jobs; always revert geometry and move on)
  timeout:          (KILL_TIMER_S + 60) × 1000 ms
    (application-level backstop; OS-level is primary — RULE-002)
  removeOnComplete: 200
  removeOnFail:     500

Background queue job options:
  attempts:         3
  backoff:          { type: 'exponential', delay: 2000 }
  removeOnComplete: 100
  removeOnFail:     200

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REDIS KEY SCHEMA (all keys owned by packages/redis/src/keys.ts)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

aeroloop:{run_id}:global_best_M
  Type: string (float serialised)
  Owner: KeepRevertAgent (GETSET on improvement), SwarmSyncAgent (GET)

aeroloop:{run_id}:node:{node_id}:solve_lock
  Type: string (node_id as value)
  TTL:  KILL_TIMER_S + 120 s
  Owner: SolverAgent (SET on start, DEL on finish/timeout)

aeroloop:{run_id}:region:{region_name}:owner
  Type: string (node_id as value)
  TTL:  REGION_TTL_S (default 600 s, renewed on heartbeat)
  Owner: SwarmSyncAgent (SETNX on claim, DEL on release)

aeroloop:{run_id}:consecutive_fails
  Type: string (integer serialised)
  Owner: KeepRevertAgent (INCR on non-KEPT, DEL on KEPT)

aeroloop:{run_id}:gep_in_progress
  Type: string ("1")
  TTL:  3600 s
  Owner: GEPOrchestrator (SET on start, DEL on complete)


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/metric.md
GLOBAL-METRIC-001 — COMPOSITE METRIC SPECIFICATION
═══════════════════════════════════════════════════════════════════════

The composite metric M is the SINGLE scalar used for all keep/revert
decisions. It is defined once per run in ExperimentRun.metric_weights
and is IMMUTABLE for the duration of the run (RULE-011 extension).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
METRIC FORMULA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Standard (buffet data available from alpha sweep):

  M = w_ld     × norm(L/D)
    + w_buffet  × norm(buffet_margin_deg)
    + w_wave    × norm(1 − Cd_wave / Cd_total)

  where norm(x) = x / x_baseline
  (ratio of current experiment value to baseline experiment value)

  buffet_margin_deg = buffet_onset_alpha − alpha_cruise
    (from ExperimentRun.operating_conditions.alpha_cruise)

Fallback (alpha sweep not run; buffet_onset_alpha is null):
  Redistribute w_buffet proportionally to w_ld and w_wave:

  w_ld_adj   = w_ld   + w_buffet × (w_ld   / (w_ld + w_wave))
  w_wave_adj = w_wave + w_buffet × (w_wave / (w_ld + w_wave))
  M = w_ld_adj × norm(L/D) + w_wave_adj × norm(1 − Cd_wave / Cd_total)

Edge case: if baseline value of any term is 0, that term is excluded
and remaining weights are renormalised to sum to 1.0. Logged as a
SANITY_WARNING event.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WEIGHT CONSTRAINTS (enforced at POST /runs — see API-002)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  All weights ≥ 0.0
  Sum of weights = 1.0 ± 0.001
  w_ld ≥ 0.3   (L/D must always be a meaningful component)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WAVE DRAG COMPUTATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Primary method (SU2 far-field decomposition — use if available):
  Cd_wave = Cd_total − Cd_induced − Cd_viscous

Fallback (far-field decomposition unavailable in solver output):
  Cd_induced ≈ CL² / (π × AR × e)
    e (Oswald efficiency) = 0.85 default; configurable in solver_config
  Cd_viscous ≈ Cf × (1 + 1.2×(t/c) + 100×(t/c)⁴) × S_wet/S_ref
    Cf = flat-plate skin friction coefficient from operating conditions
    t/c = mean thickness/chord ratio (from wing.geo at runtime)
    S_wet, S_ref = from geometry (computed by GMSH integration)
  Cd_wave = max(0, Cd_total − Cd_induced − Cd_viscous)
    (clamped to 0; negative wave drag is unphysical)

The method used must be recorded in experiment stage_timings JSONB
as "wave_drag_method": "farfield" | "fallback".

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SANITY BOUNDS (warn via SANITY_WARNING event; do NOT halt the loop)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Coefficient      Warn if outside range
─────────────────────────────────────────────────────────────────────
CL               [-0.5, 3.0]
CD               [0.001, 0.5]
L/D              [0.0, 80.0]
CMy              [-1.0, 1.0]
Cd_wave          < 0 (clamp to 0, emit warning)
buffet_margin    < 0 (cruise AoA beyond buffet onset; always warn)

Sanity warnings are logged and recorded in the experiment record.
They do not prevent KEEP/REVERT decision or block the next experiment.
Three consecutive SANITY_WARNING experiments on the same coefficient
trigger a LOOP_CONSECUTIVE_FAIL (same counter as non-KEPT outcomes).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
METRIC VERSIONING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Each ExperimentRun records metric_version = "1.0".
Any change to the formula requires a version bump AND a new run.
Old experiments are NEVER retroactively rescored.
MetricAgent validates metric_version matches its compiled formula
version on every startup; mismatch halts the loop.


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/mesh-quality.md
GLOBAL-MESH-001 — MESH QUALITY THRESHOLDS
═══════════════════════════════════════════════════════════════════════

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
QUALITY GATES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ALL gates must pass for MESH_QUALITY_PASS status.
ANY gate failing triggers MESH_QUALITY_FAIL → immediate REVERT.

Metric                     Threshold          Fail condition
──────────────────────────────────────────────────────────────────────
Min orthogonal quality     ≥ 0.15             any cell below 0.15
Max skewness               ≤ 0.85             any cell above 0.85
y+ (95th percentile)       ≤ 5.0              95th pct above 5.0
y+ mean                    0.5 – 2.0          mean outside range
Cell count                 ±30% of target     outside tolerance band
Negative volume cells      = 0                any cell with vol < 0
Max aspect ratio           ≤ 5000             any cell above 5000

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MESH LEVEL TARGETS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Level     Target cell count    BL layers    Growth rate    y+ target
──────────────────────────────────────────────────────────────────────
coarse    500k – 1.5M          20           1.20           1.0
medium    1.5M – 4M            30           1.15           1.0
fine      4M – 10M             40           1.10           1.0

Default level is set by MESH_DEFAULT_LEVEL env var (default: medium).
The genome strategy may override per-experiment (e.g., coarse for
global search; fine for refinement near best geometry).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
QUALITY REPORT FILE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MeshQualityAgent writes to .aeroloop/mesh_quality_exp_{N}.json:
  {
    "experiment_n":            144,
    "level":                   "medium",
    "passed":                  true,
    "metrics": {
      "min_orthogonal_quality":  0.31,
      "max_skewness":            0.71,
      "y_plus_95th":             1.8,
      "y_plus_mean":             0.94,
      "cell_count":              2140000,
      "cell_count_target":       2000000,
      "cell_count_tolerance_pct": 30,
      "negative_volume_cells":   0,
      "max_aspect_ratio":        1240
    },
    "failures": []
  }

This file is gitignored (not committed); it exists for local debug.
The quality gate result (passed/failed + metrics) is stored in the
experiment DB record (mesh_quality_ok field and stage_timings JSONB).


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/environment.md
GLOBAL-ENV-001 — ENVIRONMENT & CONFIGURATION
═══════════════════════════════════════════════════════════════════════

All configuration is injected via environment variables.
NO hardcoded values anywhere in the codebase.
All variables are validated on startup via the Zod schema below.
Startup FAILS FAST with a descriptive error if any required variable
is missing or invalid. Optional variables have defaults shown.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ZOD SCHEMA (packages/types/src/env.ts — used by all apps)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

export const EnvSchema = z.object({

  // ── Core ──────────────────────────────────────────────────────────
  NODE_ENV:    z.enum(['development', 'test', 'production']),
  LOG_LEVEL:   z.enum(['debug','info','warn','error']).default('info'),

  // ── Database ──────────────────────────────────────────────────────
  DATABASE_URL:       z.string().url(),
  DATABASE_POOL_MIN:  z.coerce.number().default(2),
  DATABASE_POOL_MAX:  z.coerce.number().default(10),

  // ── Redis ─────────────────────────────────────────────────────────
  REDIS_URL:          z.string().url(),
  REDIS_KEY_PREFIX:   z.string().default('aeroloop'),

  // ── S3 ────────────────────────────────────────────────────────────
  S3_ENDPOINT:        z.string().url(),
  S3_BUCKET:          z.string().min(1),
  S3_ACCESS_KEY:      z.string().min(1),
  S3_SECRET_KEY:      z.string().min(1),
  S3_REGION:          z.string().default('us-east-1'),

  // ── Solver ────────────────────────────────────────────────────────
  SU2_BINARY:         z.string().min(1),
  KILL_TIMER_S:       z.coerce.number().min(60).max(3600).default(480),
  MPI_RANKS:          z.coerce.number().min(1).max(256).default(8),
  SU2_CONFIG_TEMPLATE_DIR: z.string().min(1),
  OSWALD_EFFICIENCY:  z.coerce.number().min(0.5).max(1.0).default(0.85),

  // ── Meshing ───────────────────────────────────────────────────────
  GMSH_PYTHON:        z.string().min(1),
  AUTO_MESH_SCRIPT:   z.string().min(1),
  MESH_DEFAULT_LEVEL: z.enum(['coarse','medium','fine']).default('medium'),
  Y_PLUS_TARGET:      z.coerce.number().default(1.0),

  // ── Geometry ──────────────────────────────────────────────────────
  GEOMETRY_FILE:      z.string().min(1).default('geometry/wing.geo'),

  // ── Git ───────────────────────────────────────────────────────────
  GIT_AUTHOR_NAME:    z.string().default('AeroLoopBot'),
  GIT_AUTHOR_EMAIL:   z.string().email().default('bot@aeroloop.internal'),

  // ── Loop behaviour ────────────────────────────────────────────────
  CONSECUTIVE_FAIL_HALT:  z.coerce.number().min(1).default(3),
  GEP_TRIGGER_INTERVAL:   z.coerce.number().min(100).default(500),
  WIKI_COMPILE_INTERVAL:  z.coerce.number().min(1).default(10),
  SURROGATE_MIN_SAMPLES:  z.coerce.number().default(50),

  // ── Swarm ─────────────────────────────────────────────────────────
  SWARM_ENABLED:          z.coerce.boolean().default(false),
  NODE_ID_FILE:           z.string().default('.aeroloop/node_id'),
  SWARM_API_TIMEOUT_MS:   z.coerce.number().default(5000),
  SWARM_REGION_TTL_S:     z.coerce.number().default(600),
  SWARM_HEARTBEAT_S:      z.coerce.number().default(30),

  // ── API server ────────────────────────────────────────────────────
  API_PORT:               z.coerce.number().default(4000),
  JWT_SECRET:             z.string().min(32),
  JWT_EXPIRY:             z.string().default('24h'),

  // ── Notifications ─────────────────────────────────────────────────
  NOTIFICATION_WEBHOOK_URL: z.string().url().optional(),
  NOTIFICATION_EMAIL:       z.string().email().optional(),

});

export type Env = z.infer<typeof EnvSchema>;

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LOCAL DEV .env FILE (template — copy to .env and fill secrets)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

NODE_ENV=development
LOG_LEVEL=debug
DATABASE_URL=postgresql://aeroloop:secret@localhost:5432/aeroloop
REDIS_URL=redis://localhost:6379
S3_ENDPOINT=http://localhost:9000
S3_BUCKET=aeroloop-results
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
SU2_BINARY=/usr/local/bin/SU2_CFD
SU2_CONFIG_TEMPLATE_DIR=packages/solver/src/config-templates
GMSH_PYTHON=/usr/local/bin/python3
AUTO_MESH_SCRIPT=packages/meshing/python/auto_mesh.py
KILL_TIMER_S=480
MPI_RANKS=8
GEOMETRY_FILE=geometry/wing.geo
GIT_AUTHOR_NAME=AeroLoopBot
GIT_AUTHOR_EMAIL=bot@aeroloop.internal
JWT_SECRET=change-me-to-at-least-32-chars-in-prod
SWARM_ENABLED=false


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/commit-format.md
GLOBAL-COMMIT-001 — GIT COMMIT FORMAT & HOOK IMPLEMENTATION
═══════════════════════════════════════════════════════════════════════

This spec defines the hook implementations that enforce RULE-060 and
RULE-061. The canonical formats are defined in CLAUDE.md; this file
contains the hook code and the S3 path conventions.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
hooks/pre-commit
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

#!/usr/bin/env bash
# Enforces RULE-001: only geometry/wing.geo may be committed by
# AeroLoopBot. Developer commits are unrestricted.

set -euo pipefail

if [ "${GIT_AUTHOR_NAME:-}" = "AeroLoopBot" ]; then
  CHANGED=$(git diff --cached --name-only)
  DISALLOWED=$(echo "$CHANGED" | grep -v '^geometry/wing\.geo$' \
                                       -v '^\\.aeroloop/wiki/' \
                                       -v '^\\.aeroloop/genome/' \
                                       -v '^\\.aeroloop/knowledge_graph_summary\\.md$' \
                               || true)
  if [ -n "$DISALLOWED" ]; then
    echo "ERROR: AeroLoopBot attempted to commit disallowed files:"
    echo "$DISALLOWED"
    echo "Only geometry/wing.geo, .aeroloop/wiki/*, .aeroloop/genome/*,"
    echo "and .aeroloop/knowledge_graph_summary.md are permitted."
    exit 1
  fi
fi
exit 0

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
hooks/commit-msg
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

#!/usr/bin/env bash
# Enforces RULE-060 and RULE-061: AeroLoopBot commit message format.

set -euo pipefail

if [ "${GIT_AUTHOR_NAME:-}" = "AeroLoopBot" ]; then
  FIRST_LINE=$(head -1 "$1")
  if ! echo "$FIRST_LINE" | grep -qE \
    '^(exp\([0-9]+\): M=|chore\(gep\): generation|docs\(wiki\): batch)'; then
    echo "ERROR: AeroLoopBot commit message does not match RULE-060/061."
    echo "Expected one of:"
    echo "  exp({N}): M={val} delta={+/-val} L/D={val} [KEEP] ..."
    echo "  chore(gep): generation {N} evolved at exp {N}"
    echo "  docs(wiki): batch {N}–{N} compiled | {N} articles"
    echo "Got: $FIRST_LINE"
    exit 1
  fi
fi
exit 0

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
S3 PATH CONVENTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

runs/{run_id}/exp-{experiment_n}/mesh.su2
runs/{run_id}/exp-{experiment_n}/mesh.su2.sha256
runs/{run_id}/exp-{experiment_n}/results/surface_flow.csv
runs/{run_id}/exp-{experiment_n}/results/history.csv
runs/{run_id}/exp-{experiment_n}/results/surface_flow.csv.sha256
runs/{run_id}/exp-{experiment_n}/results/history.csv.sha256
runs/{run_id}/exp-{experiment_n}/geometry_after.geo
runs/{run_id}/best/geometry_best.geo
runs/{run_id}/best/geometry_best.geo.sha256

All uploads are atomic: file is uploaded first, then its .sha256
sidecar file is written. A missing .sha256 sidecar means the upload
is incomplete and the experiment must be treated as SOLVE_FAIL.


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/logging.md
GLOBAL-LOG-001 — STRUCTURED LOGGING SPECIFICATION
═══════════════════════════════════════════════════════════════════════

All log output is structured JSON using pino.
No plain-text log statements (console.log etc.) are permitted in
production code. ESLint rule no-console enforces this.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REQUIRED FIELDS ON EVERY LOG LINE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{
  "level":        "info",
  "time":         "2025-01-15T14:32:01.123Z",
  "service":      "loop | api | cli",
  "run_id":       "uuid-v4",
  "experiment_n": 144,
  "node_id":      "uuid-v4",
  "stage":        2,
  "msg":          "Mesh quality check passed"
}

For log lines that predate a run (startup, registration), run_id,
experiment_n, and stage may be null.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REQUIRED LOG EVENTS AND CONTEXT FIELDS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Event                      Level   Required additional fields
──────────────────────────────────────────────────────────────────────
Loop start                 info    node_id, run_id, swarm_enabled
Dirty tree halt            error   dirty_files: string[]
Experiment start           info    experiment_n, mutation_parameter_id,
                                   mutation_delta, mutation_strategy
Bounds check result        info    valid, parameter_id, proposed_value,
                                   bounds_min, bounds_max
Detect-impact result       info    mesh_risk, recommendation, confidence
Geometry written           info    sha256_before, sha256_after
Mesh started               info    level, cell_count_target
Mesh quality result        info    passed, all metrics from quality report
Mesh quality fail          warn    failures: string[]
Solve started              info    config_hash, mpi_ranks, kill_timer_s
Solve timeout              warn    wall_time_s, kill_timer_s
Solve unconverged          warn    final_residual, iterations
Solve converged            info    wall_time_s, final_residual, iterations
Results uploaded           info    s3_key, sha256
Extraction complete        info    cl, cd, cmy, ld_ratio, cd_wave,
                                   buffet_onset_alpha, wave_drag_method
Sanity warning             warn    coefficient, value, expected_range
Metric computed            info    M, M_best_at_time, delta_M
KEEP decision              info    M_new, M_best, delta_M, git_commit_hash
REVERT decision            info    M_new, M_best, delta_M, reason
Consecutive fail count     warn    consecutive_fail_count, threshold
GEP start                  info    generation, experiment_n
GEP complete               info    generation, fitness_before, fitness_after,
                                   strategy_before, strategy_after
GEP failed                 error   error_message, genome_preserved: true
KG interaction discovered  info    param_a, param_b, p_value, strength,
                                   direction
Dead zone declared         warn    parameter_id, failed_attempts, region
Hot zone declared          info    parameter_id, keep_rate, region
Node registered            info    node_id, hostname, run_id
Node stale                 warn    node_id, last_heartbeat, elapsed_s
Region claimed             info    region_name, node_id, ttl_s
Region released            info    region_name, node_id
Swarm offline mode         warn    node_id, coordinator_url,
                                   accumulated_experiments: 0

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DECISION LOG (local append-only file)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

KeepRevertAgent appends one JSON line per experiment to:
  .aeroloop/decision_log.jsonl

This is a human-readable audit trail for operators. It is gitignored
(not committed). Format:

  {"experiment_n":144,"status":"KEPT","M":0.8821,"delta_M":0.0032,
   "parameter_id":"taper_ratio","delta":-0.03,"git_commit":"a1b2c3d",
   "timestamp":"2025-01-15T14:45:00.000Z"}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LOG RETENTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Development:  stdout only
Production:   ship to configured log aggregator (Loki / ELK)
              retain 90 days minimum
              KEPT and REVERT decision lines also in decision_log.jsonl


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/testing.md
GLOBAL-TEST-001 — TESTING PHILOSOPHY & PATTERNS
═══════════════════════════════════════════════════════════════════════

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PRINCIPLES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Unit tests own correctness of pure logic.
   All pure functions (metric computation, bounds checking, genome
   operators, wave drag formula, buffet detection) must have exhaustive
   unit tests with known-good values from reference aerodynamics data.

2. Integration tests own pipeline stage contracts.
   Each stage boundary is tested end-to-end with real GMSH and either
   mocked SU2 (pre-recorded output) or real SU2 on a fast reference
   geometry (NACA 0012 at coarse mesh level).

3. Git integrity tests are non-negotiable.
   Every test that exercises keep/revert must assert:
     - SHA256(wing.geo after revert) == SHA256(wing.geo before mutation)
     - KEEP commit message matches RULE-060 regex exactly
     - No uncommitted changes remain in working tree after each test

4. No test may use sleep() or wall-clock assertions for correctness.
   Use injected fake clocks / mock timers for kill-timer tests.
   Exception: testcontainer startup waits (use explicit readiness polls).

5. Test data is fixed, versioned, and committed.
   Reference meshes and solver output CSV fixtures live in:
     packages/extraction/fixtures/
     packages/meshing/fixtures/
   Never silently regenerate fixtures. Any fixture update is a
   deliberate PR with a comment explaining why values changed.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOOLING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test runner:        Vitest (all packages and apps)
DB/Redis in tests:  @testcontainers/postgresql, @testcontainers/redis
S3 in tests:        MinIO testcontainer or @aws-sdk/client-s3 mock
SU2 in unit tests:  MockSU2Runner injectable (same interface as real
                    SU2Runner; returns pre-recorded fixture outputs)
SU2 in integration: real SU2Runner with NACA 0012 at coarse level
GMSH in tests:      real GMSH; coarse mesh level only for speed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
COVERAGE THRESHOLDS (CI enforces; build fails if not met)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Package                  Line coverage threshold
──────────────────────────────────────────────────
packages/geometry        95%
packages/extraction      95%
packages/git-ops         100%
packages/gep             90%
packages/knowledge-graph 85%
packages/meshing         85%
packages/solver          85%
packages/wiki            80%
packages/surrogate       80%
All other packages       80%

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NAMING CONVENTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

packages/{name}/tests/unit/{module}.test.ts
packages/{name}/tests/integration/{module}.integration.test.ts
apps/{name}/tests/e2e/{scenario}.e2e.test.ts

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
INTEGRATION TEST SUITE (IT-001 through IT-006)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

IT-001: Single Experiment Happy Path
  Setup:    Known-good baseline .geo; pre-recorded SU2 outputs; real GMSH.
  Verifies:
  - Experiment record created with correct stage results
  - geometry_sha256_before matches pre-mutation hash
  - geometry_sha256_after matches post-mutation hash
  - Mesh uploaded to S3 with matching SHA256 sidecar
  - Solver outputs uploaded to S3 with matching SHA256 sidecar
  - Composite metric M within expected range ± 0.001
  - KEPT decision triggers git commit matching RULE-060 regex
  - ExperimentRun.best_M updated to new M

IT-002: Kill Timer Integration
  Setup:    MockSU2Runner that sleeps KILL_TIMER_S + 60 s (injected).
  Verifies:
  - OS-level timeout fires within KILL_TIMER_S + 5 s
  - BullMQ job marked failed
  - wing.geo SHA256 after revert matches pre-mutation SHA256
  - Experiment status = SOLVE_TIMEOUT
  - Loop dispatches next experiment within 30 s of timeout
  - No uncommitted changes in working tree after revert

IT-003: Mesh Quality Gate
  Setup:    GMSH configured to produce orthogonality = 0.10 (below threshold).
  Verifies:
  - MESH_QUALITY_FAIL emitted
  - No SU2 config generated; no solver started
  - Experiment status = MESH_FAIL
  - wing.geo reverted (SHA256 matches pre-mutation)

IT-004: DB Monotonicity Constraint
  Setup:    Two concurrent POST /experiments with same (run_id, experiment_n).
  Verifies:
  - Second insert returns HTTP 409
  - First record unchanged in DB
  - No duplicate rows

IT-005: GEP Trigger at n=500
  Setup:    500 experiment records seeded in DB (no actual CFD).
            experiment_n 500 dispatched.
  Verifies:
  - GEP_EVOLUTION_START emitted; core queue pauses
  - .aeroloop/genome/generation_1.json written; validates GenomeSchema
  - genome committed with RULE-061 message format
  - GEP_EVOLUTION_COMPLETE emitted
  - Loop resumes; experiment 501 dispatched

IT-006: Swarm Region Non-Collision (4 nodes)
  Setup:    4 loop instances, shared Redis + DB, shared git repo.
  Verifies:
  - At most 1 node holds each region claim at any time
    (verified by inspecting Redis state at 100 ms intervals)
  - Git log shows zero merge conflicts over 500 experiments
  - All experiment records in shared DB have distinct
    (node_id, experiment_n) tuples with no gaps per node


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-0/requirements.md
STAGE 0 — INITIALISATION
═══════════════════════════════════════════════════════════════════════

Purpose: lock the run configuration, validate the baseline geometry
end-to-end, and establish M_best before the optimisation loop starts.
A failed baseline is a hard stop: do not begin optimisation on a broken
starting point.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-0-001: Create and lock ExperimentRun
  The orchestrator calls POST /runs with all metric weights, operating
  conditions, solver config, and mesh config. The response run_id is
  stored in .aeroloop/run_id. Status transitions from INITIALISING to
  RUNNING only after the baseline experiment completes successfully.

FR-0-002: Validate baseline geometry
  Before the first mutation loop iteration:
    (a) Read geometry/wing.geo and compute SHA256.
    (b) Check all parameters in GLOBAL-GEOM-001 against hard bounds
        (BoundsCheckAgent). Any violation is a fatal baseline error.
    (c) Check cross-parameter constraints. Violations are fatal.

FR-0-003: Run baseline through full pipeline
  The baseline geometry (experiment_n = 0) must traverse the complete
  pipeline: bounds → mesh → mesh quality → solve → extract → metric.
  This is NOT a dry run. It is a real CFD solve on the baseline.
  The kill timer is active. If the baseline solve times out or fails
  to converge, Stage 0 fails and the run is never started.

FR-0-004: Record baseline as exp-0 and set M_best
  On successful baseline completion:
    (a) Insert experiment record with experiment_n = 0, status = KEPT
        (baseline is always "kept" as the starting point).
    (b) Set ExperimentRun.best_M = baseline M.
    (c) Set ExperimentRun.best_experiment_n = 0.
    (d) Write Redis key aeroloop:{run_id}:global_best_M = baseline M.
    (e) Write .aeroloop/best_metric.json:
          { "M": {value}, "experiment_n": 0, "timestamp": "..." }
    (f) Tag git commit: aeroloop-run-{run_id}-baseline
    (g) Set ExperimentRun.baseline_experiment_id.

FR-0-005: Seed knowledge graph with pre-seeded interactions
  Insert the pre-seeded interactions from GLOBAL-GEOM-001 into
  knowledge_graph_interactions with confidence = 0.3.
  These are overwritten by real data as experiments accumulate.

FR-0-006: Initialise swarm node (if SWARM_ENABLED)
  Call POST /swarm/nodes with node identity and capabilities.
  Claim an initial region before the first mutation proposal.
  If coordinator is unreachable: enter OFFLINE_MODE and proceed.

FR-0-007: Git initialisation
  Create run branch: aeroloop/run-{run_id}
  Baseline commit already exists (HEAD before loop starts).
  The pre-commit and commit-msg hooks must be installed and active.
  Verify hook installation before proceeding (halt if not found).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-0-001: Run record created and locked
  [ ] POST /runs returns 201 with a valid run_id
  [ ] ExperimentRun.status = INITIALISING immediately after creation
  [ ] Config fields (metric_weights, operating_conditions,
      solver_config, mesh_config) are identical when read back via
      GET /runs/{run_id}
  [ ] A second POST /runs from the same node_id returns 409

AC-0-002: Baseline geometry valid
  [ ] All GLOBAL-GEOM-001 parameters read from wing.geo are within
      hard bounds (any violation causes Stage 0 to fail with a clear
      error naming the violating parameter)
  [ ] All cross-parameter constraints pass
  [ ] SHA256 of wing.geo is computed and stored

AC-0-003: Baseline pipeline completes
  [ ] Baseline mesh passes all GLOBAL-MESH-001 quality gates
  [ ] Baseline solve converges within KILL_TIMER_S
  [ ] CL, CD, CMy, L/D extracted without NaN or null values
  [ ] Composite M computed; no sanity warnings on baseline (if warnings
      present they are logged but Stage 0 still proceeds)

AC-0-004: M_best initialised correctly
  [ ] experiment record with experiment_n=0, status=KEPT in DB
  [ ] ExperimentRun.best_M == baseline M (within float precision)
  [ ] Redis aeroloop:{run_id}:global_best_M == baseline M
  [ ] .aeroloop/best_metric.json exists with correct M value
  [ ] Git tag aeroloop-run-{run_id}-baseline present on HEAD

AC-0-005: KG seeded
  [ ] 5 pre-seeded interaction records in knowledge_graph_interactions
      with confidence=0.3 and the correct param_a/param_b pairs

AC-0-006: Hooks active
  [ ] hooks/pre-commit exists and is executable
  [ ] hooks/commit-msg exists and is executable
  [ ] A test commit by AeroLoopBot touching a disallowed file is
      rejected by pre-commit hook (verified in IT-001)


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-1/requirements.md
STAGE 1 — GEOMETRY MUTATION
═══════════════════════════════════════════════════════════════════════

Purpose: propose exactly one informed geometry change, validate it
against bounds and physics, and write it to wing.geo.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-1-001: Read context before proposing
  GeometryMutationAgent MUST read ALL of the following before
  generating a mutation proposal:
    (a) geometry/wing.geo (current parameter values)
    (b) Last 10 git log entries (via git log --oneline -10)
    (c) .aeroloop/best_metric.json
    (d) GET /knowledge-graph/summary?run_id={run_id}&region={claimed_region}
        (with 3 s timeout and local cache fallback per GLOBAL-API-001)
    (e) Current genome from ExperimentRun.current_genome
    (f) Relevant wiki articles (GET /wiki/articles?run_id={run_id}
        &parameter_id={parameter_id} for the parameter being considered)

FR-1-002: Propose exactly one mutation
  The proposal must be a single JSON object:
    {
      "parameter_id":   "taper_ratio",
      "current_value":  0.41,
      "proposed_value": 0.38,
      "delta":          -0.03,
      "strategy":       "fine_search",
      "rationale":      "KG sensitivity 0.61; last keep at -0.02 delta"
    }
  Compound mutations (changing more than one parameter) are forbidden
  in normal mode. They are only permitted when the genome strategy is
  "global" and topology_mutation = true AND the genome's
  topology_probability > 0.

FR-1-003: Anti-repetition check
  The agent MUST compare the proposed (parameter_id, proposed_value)
  against the last 10 experiments from the DB. If an identical or
  near-identical mutation was already attempted (|delta_proposed -
  delta_past| < 0.001 × parameter_range), the agent MUST:
    (a) Emit MUTATION_SKIPPED_DUPLICATE
    (b) Propose a different parameter or a different delta magnitude
  If the agent cannot find a non-duplicate proposal after 5 attempts,
  it escalates the strategy (e.g., from fine_search to medium_search)
  and tries again.

FR-1-004: Dead zone avoidance
  Before proposing, the agent checks the KG summary for dead zones.
  A dead zone exists for a parameter when: ≥ 3 experiments targeting
  that parameter all resulted in non-KEPT outcomes (REVERT, TIMEOUT,
  MESH_FAIL) with no KEPT outcome.
  If the proposed parameter is in a dead zone:
    (a) Emit MUTATION_SKIPPED_DEAD_ZONE
    (b) Choose a different parameter from the hot zone list or from
        unconstrained parameters

FR-1-005: detect-impact call
  After choosing a proposal but BEFORE writing wing.geo, the agent
  calls POST /knowledge-graph/detect-impact.
  The response is logged with all fields.
  If mesh_risk = HIGH: log warning; agent may skip (must log decision).
  If recommendation = SKIP: log; agent may skip (must log decision).
  Skipping is advisory; it is never mandatory.

FR-1-006: Bounds validation (BoundsCheckAgent, mandatory)
  After the proposal is accepted (detect-impact consulted), pass the
  proposed value to BoundsCheckAgent:
    (a) Check proposed_value against GLOBAL-GEOM-001 hard bounds
    (b) Check all cross-parameter constraints with the proposed value
        applied
  On any failure: emit BOUNDS_INVALID, record experiment as BOUNDS_FAIL,
  revert (no file written), return to step FR-1-001 for this experiment.
  On success: emit BOUNDS_VALID.

FR-1-007: Write wing.geo (ONLY on bounds pass)
  Compute SHA256 of wing.geo before write.
  Write the proposed_value for the parameter_id into wing.geo using
  packages/geometry/src/wing-geo-parser.ts.
  Compute SHA256 of wing.geo after write.
  Emit GEOMETRY_WRITTEN with both SHA256 values.
  The write is the point of no return; the Stage 5 revert path
  (git checkout geometry/wing.geo) is now active.

FR-1-008: Swarm region guard (if SWARM_ENABLED)
  Before writing wing.geo, verify that the node still holds its
  Redis region claim (TTL > 0). If the claim has expired:
    (a) Attempt to renew it via SETNX.
    (b) If renewal fails (another node claimed it): do NOT write
        wing.geo; re-claim a different region; re-propose.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-1-001: Context reading
  [ ] In a 100-experiment integration test, every MUTATION_PROPOSAL
      log entry contains non-null rationale referencing KG sensitivity
      score or wiki article experiment number
  [ ] If KG API times out (simulated), agent uses cached summary
      (verified: response time > 3 s → WARN log + cached data used)

AC-1-002: Single mutation only
  [ ] In 1000-experiment run, zero experiments have more than one
      parameter changed in wing.geo vs HEAD (verified by git diff)
  [ ] Compound mutation attempt with strategy=fine_search and
      topology_mutation=false is rejected and logged

AC-1-003: Anti-repetition
  [ ] No two consecutive experiments mutate the same parameter with
      |delta_proposed - delta_past| < 0.001 × parameter_range
  [ ] On 5 failed anti-repetition attempts, strategy escalation
      event is emitted and logged

AC-1-004: Dead zone avoidance
  [ ] A parameter declared as dead zone (3+ fails, 0 keeps) is not
      targeted for 10 subsequent experiments (verified from DB)
  [ ] MUTATION_SKIPPED_DEAD_ZONE emitted and logged when avoided

AC-1-005: Bounds validation
  [ ] A mutation proposing a value 0.001 outside hard bounds is
      rejected; experiment recorded as BOUNDS_FAIL; wing.geo unchanged
  [ ] A cross-parameter violation (tip_chord > root_chord) is caught
      and recorded as BOUNDS_FAIL; wing.geo unchanged
  [ ] BOUNDS_VALID emitted on every experiment that proceeds past
      this stage

AC-1-006: Geometry written correctly
  [ ] SHA256 before != SHA256 after for every non-BOUNDS_FAIL experiment
  [ ] The changed parameter value in the written wing.geo equals
      proposed_value (parsed and verified programmatically)
  [ ] GEOMETRY_WRITTEN emitted with both SHA256 values


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-2/requirements.md
STAGE 2 — MESH GENERATION
═══════════════════════════════════════════════════════════════════════

Purpose: generate a high-quality CFD mesh from the mutated wing.geo,
validate it against GLOBAL-MESH-001 thresholds, and upload it to S3.
A mesh quality failure is a cheap early abort that prevents a
garbage-in/garbage-out solve.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-2-001: Select mesh level
  The mesh level (coarse / medium / fine) is determined in order of
  precedence:
    (1) Genome strategy override (genome.mesh_level if set)
    (2) MESH_DEFAULT_LEVEL environment variable
    (3) Default: medium
  The selected level is logged and stored in stage_timings.

FR-2-002: Compute first-cell height for y+ target
  Before calling GMSH, compute the boundary layer first-cell height:
    Δy = y+ × ν / u_τ
    u_τ = sqrt(τ_w / ρ)
    τ_w estimated from flat-plate Cf at operating Reynolds number
  Use Y_PLUS_TARGET env var (default 1.0).
  Pass Δy to auto_mesh.py as --first_cell_height argument.

FR-2-003: Run GMSH via auto_mesh.py
  Execute:
    {GMSH_PYTHON} {AUTO_MESH_SCRIPT}
      --geo geometry/wing.geo
      --level {level}
      --first_cell_height {Δy}
      --output /tmp/aeroloop/exp-{N}/mesh.su2
  The subprocess must have a timeout of 4 × KILL_TIMER_S (meshing
  should never take longer than the solve). On timeout: MESH_FAIL.
  auto_mesh.py must:
    - Select the appropriate .geo template for the level
    - Apply curvature-based surface sizing
    - Apply boundary layer fields with the computed Δy and growth rate
    - Export in SU2 format

FR-2-004: Validate mesh quality (MeshQualityAgent)
  After GMSH completes, MeshQualityAgent parses the mesh file and
  checks ALL gates in GLOBAL-MESH-001. ALL must pass.
  Writes quality report to .aeroloop/mesh_quality_exp_{N}.json.
  Emits MESH_QUALITY_PASS or MESH_QUALITY_FAIL.
  On MESH_QUALITY_FAIL: geometry reverted, experiment = MESH_FAIL,
  next experiment begins.

FR-2-005: Upload mesh to S3
  Upload /tmp/aeroloop/exp-{N}/mesh.su2 to:
    runs/{run_id}/exp-{experiment_n}/mesh.su2
  Compute SHA256 of the local file before upload.
  Upload the .sha256 sidecar file immediately after.
  Emit MESH_UPLOADED with s3_key and sha256.
  Store mesh_s3_key and mesh_sha256 in experiment staging data.

FR-2-006: Verify S3 upload integrity
  After upload, download the .sha256 sidecar and compare with local
  SHA256. Mismatch = MESH_FAIL (corrupted upload).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-2-001: Level selection
  [ ] Genome mesh_level override is used when present in genome
  [ ] MESH_DEFAULT_LEVEL env var used when genome has no override
  [ ] Level logged in every MESH_STARTED event

AC-2-002: y+ targeting
  [ ] Computed Δy logged with u_τ and Cf values
  [ ] In integration test, generated mesh y+ mean is within
      Y_PLUS_TARGET ± 0.5 for a reference NACA 0012 at Mach 0.78

AC-2-003: Mesh generation
  [ ] GMSH subprocess completes without error for a valid wing.geo
  [ ] GMSH timeout fires and records MESH_FAIL when mesh takes longer
      than 4 × KILL_TIMER_S (simulated with a hanging mock)
  [ ] Output mesh.su2 file exists and is non-empty

AC-2-004: Quality gate
  [ ] All GLOBAL-MESH-001 thresholds checked; all must be in quality report
  [ ] A mesh with min_orthogonal_quality = 0.10 results in MESH_QUALITY_FAIL
  [ ] A mesh with 1 negative volume cell results in MESH_QUALITY_FAIL
  [ ] A passing mesh results in MESH_QUALITY_PASS and pipeline continues

AC-2-005: S3 upload and integrity
  [ ] mesh.su2 and mesh.su2.sha256 both exist at the expected S3 path
  [ ] Downloaded sidecar SHA256 matches locally computed SHA256
  [ ] A simulated upload corruption (sidecar mismatch) results in
      MESH_FAIL and geometry revert


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-3/requirements.md
STAGE 3 — CFD SOLVE
═══════════════════════════════════════════════════════════════════════

Purpose: run SU2 on the validated mesh with dual-layer kill-timer
enforcement, check convergence, and upload results to S3.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-3-001: Acquire solve lock
  Before starting SU2, acquire the exclusive solve lock:
    Redis SETNX aeroloop:{run_id}:node:{node_id}:solve_lock {node_id}
    TTL = KILL_TIMER_S + 120 s
  If the lock cannot be acquired within 10 s: SOLVE_FAILED.
  Release the lock (DEL) in all exit paths (converged, timeout, fail).

FR-3-002: Generate SU2 configuration
  Render the SU2 .cfg file from the template at
  SU2_CONFIG_TEMPLATE_DIR for the experiment's operating conditions.
  Template variables:
    MACH_NUMBER, REYNOLDS_NUMBER, AoA (alpha_cruise),
    CFL_NUMBER, MAX_ITER, CONVERGENCE_CAUCHY_EPS,
    MESH_FILENAME, SOLUTION_FILENAME, TURBULENCE_MODEL
  Write config to /tmp/aeroloop/exp-{N}/su2_config.cfg.
  Compute SHA256 of config and log as config_hash.

FR-3-003: Run SU2 with dual kill timers (RULE-002)
  Layer 1 (OS, primary):
    timeout {KILL_TIMER_S} mpirun -n {MPI_RANKS} {SU2_BINARY}
    /tmp/aeroloop/exp-{N}/su2_config.cfg
  Layer 2 (BullMQ, backstop):
    BullMQ job timeout = (KILL_TIMER_S + 60) × 1000 ms
  Both layers MUST be active. If Layer 1 fires: emit SOLVE_TIMEOUT,
  release solve lock, revert geometry, record SOLVE_TIMEOUT.
  If Layer 2 fires (Layer 1 failed for any reason): same outcome.

FR-3-004: Check convergence
  After solve completes (without timeout), check:
    (a) Residual drop: final residual < initial residual × 10^{-convergence_cauchy_eps}
        or the configured Cauchy convergence criterion is met
    (b) Force coefficient stabilisation: |CL_{final 100 iter} -
        CL_{final 200 iter}| / |CL_{final 100 iter}| < 0.01
  If (a) OR (b) fails: emit SOLVE_UNCONVERGED.
  An unconverged solve is treated as SOLVE_FAIL:
  geometry reverted; no extraction; next experiment.

FR-3-005: Upload results to S3
  Upload to:
    runs/{run_id}/exp-{experiment_n}/results/surface_flow.csv
    runs/{run_id}/exp-{experiment_n}/results/history.csv
  And their .sha256 sidecar files (one per output file).
  Emit RESULTS_UPLOADED with s3_keys and sha256 values.
  Verify sidecar integrity immediately after upload (FR-2-006 pattern).

FR-3-006: Optional alpha sweep (buffet detection)
  If the genome strategy includes "alpha_sweep" OR if
  ExperimentRun.operating_conditions.buffet_sweep_enabled = true:
    Run additional solves at alpha = alpha_cruise ± [0.5, 1.0, 1.5, 2.0]°
    Each alpha point has its OWN kill timer (Layer 1 + Layer 2).
    Timeout on any alpha point: skip that point, continue sweep.
    Upload surface_flow_{alpha}.csv for each converged alpha point.
  This is background to the primary solve and does NOT affect the
  primary SOLVE_CONVERGED / SOLVE_TIMEOUT status.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-3-001: Solve lock
  [ ] Solve lock acquired before SU2 starts; held until after upload
  [ ] Lock released in all exit paths: converged, timeout, fail
  [ ] Attempting to acquire lock while already held on same node
      results in SOLVE_FAILED with a clear error message
  [ ] Lock TTL set to KILL_TIMER_S + 120 s

AC-3-002: Config generation
  [ ] Generated .cfg contains correct MACH_NUMBER, REYNOLDS_NUMBER,
      AoA matching ExperimentRun.operating_conditions
  [ ] config_hash logged in SOLVE_STARTED event
  [ ] Two experiments with identical operating conditions produce
      identical config SHA256 (deterministic templating)

AC-3-003: Dual kill timer (IT-002 covers integration; unit test here)
  [ ] Layer 1 (OS timeout) fires within KILL_TIMER_S + 5 s
  [ ] Layer 2 BullMQ timeout set to (KILL_TIMER_S + 60) × 1000 ms
      (verified by inspecting BullMQ job config in unit test)
  [ ] On SOLVE_TIMEOUT: solve lock released, geometry reverted,
      experiment status = SOLVE_TIMEOUT

AC-3-004: Convergence check
  [ ] A solve with residual drop < 1 order of magnitude results in
      SOLVE_UNCONVERGED and geometry revert
  [ ] A solve meeting both residual and force stabilisation criteria
      results in SOLVE_CONVERGED and pipeline continues
  [ ] CL oscillation test: inject history.csv with CL oscillating
      > 1% in last 100 vs 200 iterations → SOLVE_UNCONVERGED

AC-3-005: Results upload and integrity
  [ ] surface_flow.csv and history.csv uploaded with SHA256 sidecars
  [ ] Sidecar integrity verification passes for valid uploads
  [ ] Simulated sidecar mismatch results in SOLVE_FAIL and revert

AC-3-006: Alpha sweep (if enabled)
  [ ] Each alpha point has its own kill timer
  [ ] Timeout on one alpha point does not abort the primary solve
      status or the remaining alpha points
  [ ] Surface files for converged alpha points uploaded with sidecars


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-4/requirements.md
STAGE 4 — RESULTS EXTRACTION + METRIC
═══════════════════════════════════════════════════════════════════════

Purpose: parse SU2 outputs, compute aerodynamic coefficients,
compute the composite metric M, and prepare the complete experiment
record for Stage 5.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-4-001: Download and verify solver outputs
  Download surface_flow.csv and history.csv from S3.
  Verify each file's SHA256 against its sidecar.
  Mismatch: EXTRACTION_FAILED → geometry revert.

FR-4-002: Extract aerodynamic coefficients
  Parse surface_flow.csv using surface integration (SU2 output format):
    CL    — total lift coefficient
    CD    — total drag coefficient
    CMy   — pitching moment coefficient about y-axis
  Parse history.csv for residual history and final iteration values.
  On parse error (malformed CSV, missing columns): EXTRACTION_FAILED.

FR-4-003: Compute derived quantities
  L/D ratio:  ld_ratio = CL / CD
  Cd_wave:    using GLOBAL-METRIC-001 wave drag computation
              (far-field method if available; fallback otherwise)
              Log wave_drag_method used.
  Cd_wave clamped to 0 if negative; SANITY_WARNING emitted.

FR-4-004: Buffet onset detection (if alpha sweep was run)
  For each converged alpha point's surface_flow_{alpha}.csv:
    (a) Extract CL at that alpha
    (b) Fit CL vs alpha slope
    (c) Buffet onset = alpha where slope drops below 50% of
        linear (pre-stall) slope
  buffet_onset_alpha = detected onset alpha (null if insufficient points
  or alpha sweep not run)
  buffet_margin_deg = buffet_onset_alpha − alpha_cruise

FR-4-005: Run sanity checks
  Check all coefficients against GLOBAL-METRIC-001 sanity bounds.
  Any out-of-range value: emit SANITY_WARNING with coefficient name,
  value, and expected range. Do NOT halt; pipeline continues.

FR-4-006: Compute composite metric M
  Call MetricAgent with:
    - all extracted coefficients
    - ExperimentRun.metric_weights
    - ExperimentRun.operating_conditions.alpha_cruise
    - baseline values (from exp-0 record in DB)
  MetricAgent applies GLOBAL-METRIC-001 formula exactly.
  Returns M, the weight redistribution used, and which formula
  variant was applied (standard vs fallback).
  Emit METRICS_COMPUTED with M, M_best_at_time, delta_M.

FR-4-007: Verify provenance (RULE-005)
  Before passing results to Stage 5, verify:
    (a) experiment record will contain: run_id, experiment_n, node_id
    (b) mesh_sha256 matches the sidecar downloaded in Stage 2
    (c) results_sha256 matches the sidecar downloaded in FR-4-001
  If any provenance check fails: EXTRACTION_FAILED → revert.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-4-001: Download and verify
  [ ] SHA256 mismatch on surface_flow.csv results in EXTRACTION_FAILED
      and geometry revert (unit test with fixture)
  [ ] Both files downloaded before any parsing begins

AC-4-002: Coefficient extraction
  [ ] CL, CD, CMy extracted from reference fixture surface_flow.csv
      match known-good values within 0.0001 (unit test)
  [ ] Malformed CSV (missing header column) results in EXTRACTION_FAILED
  [ ] Missing surface_flow.csv in S3 results in EXTRACTION_FAILED

AC-4-003: Derived quantities
  [ ] ld_ratio = CL / CD verified against reference values (unit test)
  [ ] Cd_wave computed via far-field method when SU2 far-field
      columns present in surface_flow.csv; "farfield" logged
  [ ] Cd_wave computed via fallback when far-field columns absent;
      "fallback" logged
  [ ] Negative Cd_wave clamped to 0 and SANITY_WARNING emitted

AC-4-004: Buffet detection
  [ ] With 4 alpha sweep points (fixture), buffet_onset_alpha matches
      known-good value within 0.1°
  [ ] With < 3 converged alpha points, buffet_onset_alpha = null
      and M computed via fallback weight redistribution

AC-4-005: Sanity checks
  [ ] CL = 5.0 (out of range) triggers SANITY_WARNING but does not
      halt pipeline; experiment continues to Stage 5
  [ ] Three consecutive SANITY_WARNING experiments on the same
      coefficient trigger LOOP_CONSECUTIVE_FAIL counter increment

AC-4-006: Metric computation
  [ ] M computed for a reference experiment matches expected value
      within 0.001 using baseline M values from DB (unit test)
  [ ] Fallback weight redistribution sums to 1.0 ± 0.001
  [ ] METRICS_COMPUTED event contains M, M_best_at_time, delta_M

AC-4-007: Provenance
  [ ] SHA256 mismatch between stored mesh_sha256 and re-verified
      sidecar results in EXTRACTION_FAILED (unit test)
  [ ] All three provenance fields present in every experiment record


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-5/requirements.md
STAGE 5 — KEEP / REVERT DECISION
═══════════════════════════════════════════════════════════════════════

Purpose: apply the Beat-the-Best law (RULE-008), persist the complete
experiment record, and leave the git working tree in a clean state
ready for the next experiment.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-5-001: Apply Beat-the-Best decision rule
  Fetch M_best from ExperimentRun.best_M (DB; authoritative).
  In swarm mode, also fetch aeroloop:{run_id}:global_best_M from Redis.
  M_best = max(ExperimentRun.best_M, redis_global_best_M).

  If M_new > M_best:
    → KEEP path (FR-5-002)
  Else:
    → REVERT path (FR-5-003)

  Strict greater-than only. Ties (M_new == M_best) go to REVERT.

FR-5-002: KEEP path
  (a) git add geometry/wing.geo
  (b) git commit -m "{message matching RULE-060 exactly}"
      using GIT_AUTHOR_NAME=AeroLoopBot GIT_AUTHOR_EMAIL env vars
  (c) Record git_commit_hash from the commit output
  (d) Update ExperimentRun in DB:
        best_M = M_new
        best_experiment_n = experiment_n
        best_geometry_sha256 = geometry_sha256_after
        kept_count += 1
        consecutive_fail_count = 0
  (e) Update Redis: GETSET aeroloop:{run_id}:global_best_M M_new
      (only if M_new > current Redis value)
  (f) Overwrite .aeroloop/best_metric.json with new values
  (g) Copy geometry/wing.geo to S3:
        runs/{run_id}/best/geometry_best.geo (+ SHA256 sidecar)
  (h) Emit EXPERIMENT_KEPT with M_new, M_best, delta_M, git_commit_hash
  (i) Emit GLOBAL_BEST_UPDATED if Redis value was updated

FR-5-003: REVERT path (all non-KEPT outcomes including KEPT-miss,
          BOUNDS_FAIL, MESH_FAIL, SOLVE_TIMEOUT, SOLVE_FAIL,
          EXTRACT_FAIL, ABORTED)
  (a) git checkout geometry/wing.geo
      (restores to HEAD; no commit; no git revert)
  (b) Verify SHA256(wing.geo) == geometry_sha256_before
      If mismatch: emit LOOP_HALT_DIRTY_TREE; halt loop
  (c) Update ExperimentRun in DB:
        reverted_count += 1  (or timeout_count, mesh_fail_count)
        consecutive_fail_count += 1
  (d) Delete Redis solve lock (safety; already released in Stage 3
      but defensive cleanup here)
  (e) Emit EXPERIMENT_REVERTED with M_new, M_best, delta_M, reason

FR-5-004: Insert ExperimentRecord (APPEND-ONLY, RULE-011)
  In BOTH keep and revert paths, call POST /experiments with the
  complete experiment record assembled across Stages 1–5.
  This is the SINGLE INSERT for the experiment. No prior partial
  inserts have been made. All stage results are included.
  On 409 (duplicate): emit LOOP_HALT; this is a data integrity error.
  On 422 (monotonicity violation): emit LOOP_HALT.

FR-5-005: Consecutive failure check (RULE-009)
  After updating consecutive_fail_count in DB:
  If consecutive_fail_count >= CONSECUTIVE_FAIL_HALT:
    (a) PATCH /runs/{run_id} with status = PAUSED
    (b) Emit LOOP_CONSECUTIVE_FAIL
    (c) Send notification (webhook/email if configured)
    (d) Halt loop; await operator resume command

FR-5-006: Dirty tree guard (start of every experiment, not just Stage 5)
  This check is performed by LoopOrchestrator BEFORE dispatching
  Stage 1 for each new experiment:
    git status --porcelain
  If any output is returned: emit LOOP_HALT_DIRTY_TREE; halt loop.
  This prevents compounding errors from a previous failed revert.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-5-001: Beat-the-Best correctness
  [ ] M_new = 0.90, M_best = 0.88 → KEPT (unit test)
  [ ] M_new = 0.88, M_best = 0.88 → REVERTED (tie; unit test)
  [ ] M_new = 0.87, M_best = 0.88 → REVERTED (unit test)
  [ ] In swarm mode, M_best = max(DB value, Redis value)
      (unit test with DB=0.88, Redis=0.89 → M_best=0.89)

AC-5-002: KEEP path correctness
  [ ] wing.geo committed with message matching RULE-060 regex exactly
  [ ] git_commit_hash non-null in experiment record
  [ ] ExperimentRun.best_M updated to M_new in DB
  [ ] .aeroloop/best_metric.json M value equals M_new
  [ ] geometry_best.geo uploaded to S3 with SHA256 sidecar
  [ ] EXPERIMENT_KEPT event emitted with all required fields
  [ ] consecutive_fail_count reset to 0 in DB

AC-5-003: REVERT path correctness
  [ ] git checkout geometry/wing.geo executed (not git revert HEAD)
  [ ] SHA256(wing.geo) after checkout == geometry_sha256_before
      (verified in IT-002 and as a unit test with fixture .geo)
  [ ] No new commit in git log after REVERT
  [ ] ExperimentRun.reverted_count incremented
  [ ] EXPERIMENT_REVERTED event emitted with reason field

AC-5-004: Experiment record inserted once
  [ ] Exactly one DB row per experiment_n per run_id
  [ ] All stage result fields populated (null only when stage was
      short-circuited by a failure before that stage)
  [ ] status field matches the actual outcome
  [ ] 409 from POST /experiments results in LOOP_HALT (not silent ignore)

AC-5-005: Consecutive failure pause
  [ ] After CONSECUTIVE_FAIL_HALT consecutive non-KEPT outcomes,
      run status = PAUSED in DB
  [ ] LOOP_CONSECUTIVE_FAIL event emitted
  [ ] Notification webhook called (if NOTIFICATION_WEBHOOK_URL set)
  [ ] Loop does not dispatch the next experiment until operator resume

AC-5-006: Dirty tree guard
  [ ] A dirty working tree (simulated by leaving a modification in
      wing.geo without committing) halts the loop before Stage 1
      of the next experiment
  [ ] LOOP_HALT_DIRTY_TREE event emitted with list of dirty files
  [ ] Loop does not attempt mutation while dirty
Part 2 is complete. It covers:

✅ GLOBAL-API-001 — Full API contracts for all 7 endpoint groups (Health, Runs, Experiments, Knowledge Graph, Wiki, Swarm, Operator) with request/response schemas, validation rules, and error contracts
✅ GLOBAL-MSG-001 — Complete MessageType enum (all 9 stages, 47 message types), Stage enum, LoopMessage base type, BullMQ queue layout, and Redis key schema
✅ GLOBAL-METRIC-001 — Full metric formula (standard + fallback), weight constraints, wave drag computation (primary + fallback), sanity bounds, metric versioning
✅ GLOBAL-MESH-001 — Complete quality gate table, mesh level targets, quality report file format
✅ GLOBAL-ENV-001 — Complete Zod environment schema with all variables, types, defaults, and local dev .env template
✅ GLOBAL-COMMIT-001 — Hook implementation code for pre-commit and commit-msg; S3 path conventions
✅ GLOBAL-LOG-001 — Required fields on every log line, full event table with required context fields, decision log format, retention policy
✅ GLOBAL-TEST-001 — Testing principles, tooling, coverage thresholds, naming convention, full IT-001–IT-006 integration test specs
✅ Stage 0 — Full FR + AC (run init, baseline pipeline, M_best seeding, git setup, KG seeding, hook verification)
✅ Stage 1 — Full FR + AC (context reading, single mutation law, anti-repetition, dead zone avoidance, detect-impact, bounds validation, geometry write, swarm region guard)
✅ Stage 2 — Full FR + AC (level selection, y+ targeting, GMSH pipeline, quality gate, S3 upload + integrity)
✅ Stage 3 — Full FR + AC (solve lock, config generation, dual kill timers, convergence check, results upload, alpha sweep)
✅ Stage 4 — Full FR + AC (download + verify, coefficient extraction, derived quantities, buffet detection, sanity checks, metric computation, provenance)
✅ Stage 5 — Full FR + AC (beat-the-best decision, KEEP path, REVERT path using git checkout not git revert, append-only record insert, consecutive failure pause, dirty tree guard)





Part 3 of 3 — covering Stages 6 through 9, infrastructure, the operator runbook, open items, and the changelog.

Markdown

═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-6/requirements.md
STAGE 6 — WIKI COMPILATION
═══════════════════════════════════════════════════════════════════════

Purpose: every 10 experiments, compile a structured, human-readable
knowledge base from the experiment history. The wiki feeds back into
the mutation agent (FR-1-001) so the system can teach itself what
has worked and what has not. Articles must be specific: vague
generalisations are a bug.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EXECUTION MODEL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Stage 6 runs in the background queue (aeroloop-{run_id}-background).
It is NON-BLOCKING with respect to the core loop. The next mutation
proposal (Stage 1 of experiment N+1) begins immediately after Stage 5
completes; it does not wait for Stage 6.

Trigger: experiment_n % WIKI_COMPILE_INTERVAL === 0
  (WIKI_COMPILE_INTERVAL default: 10)

If a Stage 6 job is already running when the trigger fires (slow
compile), skip this trigger cycle and log a warning. Do not queue
a second concurrent compile.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-6-001: Emit compile start event
  Emit WIKI_COMPILE_STARTED with run_id, experiment_n,
  batch_start (experiment_n - WIKI_COMPILE_INTERVAL + 1),
  batch_end (experiment_n).

FR-6-002: Compile sensitivity articles (one per parameter touched)
  For each parameter_id that appeared in at least one mutation in the
  current batch (last WIKI_COMPILE_INTERVAL experiments):
    (a) Query knowledge_graph_entries for all historical entries for
        this parameter_id in this run (not just the batch)
    (b) Compute: keep_count, revert_count, keep_rate, mean_delta_M
        on keep, mean_delta_M on revert, best single-experiment delta_M
    (c) Compute direction of sensitivity: positive if mean_delta on
        positive delta_value is higher than on negative delta_value
    (d) Write article to:
          .aeroloop/wiki/sensitivity_{parameter_id}.md
        Article MUST contain:
          - parameter_id and display name
          - total sample count
          - keep rate as a percentage
          - mean delta_M on keep (to 4 decimal places)
          - best single-experiment delta_M (to 4 decimal places)
          WITH the experiment_n of that best experiment cited inline
          - direction of sensitivity with confidence score
          - dead zone status (yes/no) with reason if yes
          - hot zone status (yes/no) with threshold
          - list of the top-3 KEPT experiments for this parameter:
              exp-{N}: {parameter}={old}→{new}, delta_M={+val}
          - list of the 3 worst REVERTED experiments:
              exp-{N}: {parameter}={old}→{new}, delta_M={-val}
        Article MUST NOT contain vague text like "this parameter
        generally improves performance". Every claim must cite
        an experiment number or a computed statistic.

FR-6-003: Compile batch summary article
  One article covering the entire batch (all WIKI_COMPILE_INTERVAL
  experiments):
    File: .aeroloop/wiki/batch_summary_exp{start}_{end}.md
    Must contain:
      - batch range (experiment_n start and end)
      - total experiments: kept / reverted / timeout / mesh_fail
      - best M achieved in batch (with experiment_n)
      - worst M in batch (with experiment_n)
      - parameters attempted (ranked by keep_rate descending)
      - parameter with highest mean delta_M on keep
      - any new dead zones declared in this batch
      - any new hot zones declared in this batch
      - any new interactions discovered in this batch (from Stage 7)
      - current best M overall (all experiments to date, with exp_n)

FR-6-004: Compile interaction articles (when new interactions detected)
  After each batch, query knowledge_graph_interactions for any
  interaction with discovered_at_exp in the current batch range.
  For each new interaction:
    File: .aeroloop/wiki/interaction_{param_a}_{param_b}.md
    Must contain:
      - param_a and param_b display names
      - interaction_strength and direction (synergistic / antagonistic)
      - p_value and sample_count
      - experiment_n at which the interaction was discovered
      - a concrete example: the experiment that triggered detection
      - implication for mutation strategy:
          synergistic → consider changing both in coordinated passes
          antagonistic → avoid changing both in same experiment window

FR-6-005: Compile topology articles (when topology_mutation occurred)
  For each experiment in the batch where topology_mutation = true:
    Append to .aeroloop/wiki/topology_history.md:
      ## exp-{N}: {topology_change_description}
      - geometry_sha256_before: {hash}
      - geometry_sha256_after:  {hash}
      - M_before: {M_best_at_time}
      - M_after:  {M}
      - delta_M:  {delta}
      - outcome:  KEPT / REVERTED
      - notes:    {rationale from mutation proposal}

FR-6-006: Update knowledge graph summary
  Overwrite .aeroloop/knowledge_graph_summary.md with a concise
  summary intended for the mutation agent to read (FR-1-001(d)).
  Format:
    # Knowledge Graph Summary — {run_id} — exp {N}
    ## Hot Zones (keep_rate > 40%)
      {parameter_id}: keep_rate={val}%, sensitivity={val}, direction={dir}
    ## Dead Zones (3+ fails, 0 keeps)
      {parameter_id}: {failed_attempts} failed attempts
    ## Top Interactions
      {param_a} × {param_b}: {strength:.2f} ({direction}), p={p_value:.3f}
    ## Best Parameters by mean delta_M on keep
      1. {parameter_id}: mean delta_M = +{val:.4f} (n={count})
      ...

FR-6-007: Write wiki records to DB
  For each article written, call POST /wiki/articles (via
  packages/wiki internal DB write) with all metadata fields.
  This makes articles queryable by the mutation agent via
  GET /wiki/articles.

FR-6-008: Commit wiki files
  git add .aeroloop/wiki/ .aeroloop/knowledge_graph_summary.md
  git commit -m "docs(wiki): batch {start}–{end} compiled | {n} articles"
  (matches RULE-061 commit format)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-6-001: Trigger and non-blocking behaviour
  [ ] Stage 6 fires exactly at experiment_n % 10 === 0
  [ ] Stage 1 of experiment N+1 begins before Stage 6 completes
      (verified by log timestamps in integration test)
  [ ] Concurrent Stage 6 jobs are prevented; second trigger skipped
      and logged as warning

AC-6-002: Sensitivity articles
  [ ] One article per parameter touched in the batch
  [ ] Every article contains: keep_rate, mean_delta_M, top-3 KEPT
      experiments cited by experiment_n, 3 worst REVERTED cited
  [ ] Grep for vague phrases ("generally", "usually", "tends to")
      in article body returns zero matches (enforced by unit test
      on WikiCompilerAgent output)
  [ ] Articles are overwritten (not appended) on re-compile

AC-6-003: Batch summary article
  [ ] One file per batch at correct path
  [ ] Contains kept/reverted/timeout/mesh_fail counts summing to
      WIKI_COMPILE_INTERVAL (or fewer for final partial batch)
  [ ] Parameters ranked by keep_rate descending (unit test with
      fixture experiment set)

AC-6-004: Interaction articles
  [ ] Article generated for every new interaction with
      discovered_at_exp in the batch range
  [ ] Article contains experiment_n citation and implication text
  [ ] No article generated for pre-seeded interactions
      (confidence=0.3 seed entries excluded from article trigger)

AC-6-005: Topology article
  [ ] topology_history.md appended for every topology_mutation=true
      experiment in the batch
  [ ] SHA256 before/after present; outcome correct

AC-6-006: Knowledge graph summary
  [ ] .aeroloop/knowledge_graph_summary.md updated after every
      compile (file modification timestamp > compile start time)
  [ ] Hot zones section contains only parameters with keep_rate > 40%
  [ ] Dead zones section contains only parameters with 3+ fails, 0 keeps

AC-6-007: DB records
  [ ] wiki_articles table contains one row per article written
  [ ] GET /wiki/articles?run_id={run_id} returns all articles
  [ ] GET /wiki/articles?parameter_id=span_m returns only articles
      that reference span_m in referenced_parameters

AC-6-008: Commit format
  [ ] git log shows one commit per batch compile with message matching
      docs(wiki): batch {N}–{N} compiled | {N} articles exactly


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-7/requirements.md
STAGE 7 — KNOWLEDGE GRAPH
═══════════════════════════════════════════════════════════════════════

Purpose: after every experiment, update the structured knowledge graph
with the observed parameter sensitivity, discover interactions between
parameters, track dead and hot zones, and optionally train or update
a surrogate model. The KG is the machine-readable memory that makes
every future mutation proposal smarter than the last.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EXECUTION MODEL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Stage 7 runs in the background queue (aeroloop-{run_id}-background).
It is NON-BLOCKING. It fires after EVERY experiment regardless of
outcome (KEPT, REVERTED, TIMEOUT, MESH_FAIL, BOUNDS_FAIL).

If Stage 7 fails (unhandled exception), it logs the error and retries
up to 3 times (background queue retry policy). Failure does not halt
the core loop.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-7-001: Append raw observation to knowledge_graph_entries
  Insert one row into knowledge_graph_entries for every experiment
  that reached Stage 1 (i.e., had a mutation_parameter_id):
    parameter_id, experiment_n, node_id, delta_value, delta_M,
    outcome (KEPT | REVERTED | TIMEOUT | MESH_FAIL | BOUNDS_FAIL),
    mesh_quality_ok
  BOUNDS_FAIL experiments: delta_M = 0, mesh_quality_ok = null.

FR-7-002: Update sensitivity distribution for the parameter
  After inserting the raw observation, recompute the sensitivity
  summary for the parameter from ALL historical entries:
    sensitivity_score = |mean(delta_M on KEPT) - mean(delta_M on REVERTED)|
                        normalised to [0, 1] over all parameters in run
    direction = sign of mean(delta_M × sign(delta_value)) on KEPT outcomes
    confidence = min(1.0, sample_count / 30)
      (confidence reaches 1.0 at 30 samples)
  UPSERT into knowledge_graph_sensitivities (UNIQUE on run_id, parameter_id).
  Emit KG_UPDATE_COMPLETE.

FR-7-003: Detect and declare dead zones
  A parameter is a dead zone when:
    - 3 or more consecutive experiments targeting it resulted in
      non-KEPT outcomes (any of: REVERTED, TIMEOUT, MESH_FAIL)
    - With zero KEPT outcomes in that consecutive run
  On declaring a dead zone:
    (a) UPSERT knowledge_graph_sensitivities with dead_zone = true
    (b) Emit KG_DEAD_ZONE_DECLARED with parameter_id, failed_attempts
    (c) Log warn with parameter_id and region
  Dead zone is cleared (dead_zone = false) when a KEPT outcome is
  recorded for that parameter (even if non-consecutive).

FR-7-004: Detect and declare hot zones
  A parameter is a hot zone when:
    - keep_rate > 40% over the last 10 experiments targeting it
    AND sample_count >= 5
  On declaring (or renewing) a hot zone:
    (a) UPSERT knowledge_graph_sensitivities with hot_zone = true
    (b) Emit KG_HOT_ZONE_DECLARED with parameter_id, keep_rate, region
  Hot zone is cleared when keep_rate drops below 30% in a subsequent
  10-experiment window for that parameter.

FR-7-005: Discover parameter interactions (statistical test)
  After every experiment, for the parameter that was just mutated,
  check for interactions with all other parameters that have ≥ 5
  observations each.

  Method (Mann–Whitney U test):
    Group A: delta_M values from experiments where BOTH param_i AND
             param_j were recently active (within 5 experiments of
             each other)
    Group B: delta_M values from experiments where only param_i OR
             only param_j was active
    Minimum group size: 5 each (skip test if insufficient)
    Threshold: p_value < 0.05 (two-tailed)

  On discovery (p < 0.05):
    (a) Compute interaction_strength = |mean(A) - mean(B)| normalised
    (b) Determine direction:
          synergistic if mean(A) > mean(B) (joint activity improves M)
          antagonistic if mean(A) < mean(B)
    (c) UPSERT knowledge_graph_interactions
    (d) Emit KG_INTERACTION_DISCOVERED with all fields
    (e) Do not overwrite pre-seeded interactions unless sample_count >= 10

FR-7-006: detect-impact API implementation
  POST /knowledge-graph/detect-impact (defined in GLOBAL-API-001)
  must respond within 2 s (server-side query budget).

  Implementation:
    (a) Look up sensitivity entry for the proposed parameter_id.
        If dead_zone = true → set recommendation = SKIP,
                               mesh_risk = LOW,
                               warning = "dead zone: {N} consecutive fails"
        If hot_zone = true  → set recommendation = PROCEED,
                               confidence += 0.1 (capped at 1.0)
    (b) Look up interactions for the proposed parameter_id.
        For each interaction where interaction_strength > 0.5:
          include in known_interactions list
        If any interaction has direction = antagonistic AND
        the interacting parameter was changed in the last 3 experiments:
          set mesh_risk = MEDIUM (or HIGH if strength > 0.8)
    (c) Find similar experiments:
          SELECT * FROM experiments
          WHERE run_id = ? AND mutation_parameter_id = ?
          AND ABS(mutation_delta - proposed_delta) < 0.1 × param_range
          ORDER BY ABS(mutation_delta - proposed_delta) ASC
          LIMIT 3
    (d) Compute overall recommendation:
          PROCEED            if no dead zone, mesh_risk = LOW
          PROCEED_WITH_CAUTION if mesh_risk = MEDIUM
          SKIP               if dead_zone = true OR mesh_risk = HIGH

FR-7-007: Surrogate model training (optional, after SURROGATE_MIN_SAMPLES)
  When total KEPT experiments in this run >= SURROGATE_MIN_SAMPLES
  (default 50) AND experiment_n % 50 === 0:
    Train or retrain a Gaussian Process Regression (GPR) model using
    packages/surrogate with:
      - features: all parameter values at the time of each KEPT experiment
      - target: M value of each KEPT experiment
    Persist the model to .aeroloop/surrogate/model.pkl (Python pickle).
    Emit KG_SURROGATE_TRAINED with sample_count and model_score (R²).
    The surrogate is advisory only:
      GeometryMutationAgent MAY query it via
      GET /api/v1/knowledge-graph/surrogate-predict?parameter_id=X&value=Y
      to estimate M before proposing.
      It NEVER replaces CFD as the ground truth.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-7-001: Raw observation insert
  [ ] One knowledge_graph_entries row inserted per experiment that
      reached Stage 1 (verified over 100-experiment integration test)
  [ ] BOUNDS_FAIL experiments have delta_M = 0 and mesh_quality_ok = null
  [ ] No duplicate entries for the same (run_id, experiment_n)

AC-7-002: Sensitivity update
  [ ] After 5 KEPT experiments for span_m with positive delta_value
      all improving M: direction = "positive", confidence > 0.15
  [ ] After 30 samples: confidence = 1.0 (capped; unit test)
  [ ] sensitivity_score in [0, 1] for all parameters
  [ ] UPSERT does not create duplicate rows for same (run_id, parameter_id)

AC-7-003: Dead zone detection
  [ ] 3 consecutive REVERTED outcomes for taper_ratio declares
      dead_zone = true and emits KG_DEAD_ZONE_DECLARED (unit test)
  [ ] A KEPT outcome for taper_ratio after dead zone clears dead_zone
      to false (unit test)
  [ ] 2 consecutive REVERTs do NOT trigger dead zone (boundary test)

AC-7-004: Hot zone detection
  [ ] 7 KEPT out of 10 experiments for span_m (keep_rate = 70%)
      with sample_count >= 5 declares hot_zone = true (unit test)
  [ ] KG_HOT_ZONE_DECLARED emitted with correct keep_rate value
  [ ] Hot zone cleared when keep_rate drops to 25% in subsequent window

AC-7-005: Interaction discovery
  [ ] With 10 experiments where span_m and taper_ratio both changed
      within 5 experiments of each other (fixture data), and
      mean delta_M in joint group > mean in solo group with p < 0.05:
      interaction row inserted and KG_INTERACTION_DISCOVERED emitted
  [ ] Test is NOT run when either group has < 5 samples
  [ ] Pre-seeded interaction NOT overwritten until sample_count >= 10

AC-7-006: detect-impact responses
  [ ] Dead zone parameter returns recommendation = SKIP within 2 s
  [ ] Hot zone parameter returns PROCEED with elevated confidence
  [ ] Antagonistic interaction with recent partner change returns
      MEDIUM or HIGH mesh_risk
  [ ] similar_experiments list contains experiments within
      0.1 × param_range of proposed_delta (unit test with fixture)

AC-7-007: Surrogate training
  [ ] At experiment 50 (with 50 KEPT experiments seeded), GPR model
      is trained and .aeroloop/surrogate/model.pkl written
  [ ] KG_SURROGATE_TRAINED emitted with sample_count = 50
  [ ] Model is NOT trained when KEPT count < SURROGATE_MIN_SAMPLES
  [ ] Surrogate prediction returns a finite float; does not throw


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-8/requirements.md
STAGE 8 — GEP GENOME EVOLVER
═══════════════════════════════════════════════════════════════════════

Purpose: every GEP_TRIGGER_INTERVAL experiments (default 500), pause
the loop and evolve the mutation strategy genome using a Gene
Expression Programming approach applied to the historical improvement
data. The goal is to adapt HOW the system searches, not just WHAT it
searches.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EXECUTION MODEL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Stage 8 runs in the dedicated GEP queue (aeroloop-{run_id}-gep).
It BLOCKS the core queue: no new mutation experiments are dispatched
while GEP evolution is active. Background jobs (KG, wiki, swarm sync)
continue normally during GEP.

Trigger: experiment_n % GEP_TRIGGER_INTERVAL === 0
  (GEP_TRIGGER_INTERVAL default: 500)

The trigger check occurs AFTER Stage 5 of experiment N completes.
Stage 8 runs before Stage 1 of experiment N+1 begins.

Recovery: if gep_in_progress = true on cold restart, re-run GEP from
the last completed checkpoint before resuming the loop.

Checkpoints are written after each phase:
  .aeroloop/genome/gep_checkpoint.json
  { "phase": "EVALUATED"|"SELECTED"|"EVOLVED"|"WRITTEN",
    "generation": N, "experiment_n": N, "timestamp": "..." }

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GENOME SCHEMA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

interface GeneticGenome {
  // Strategy gene: controls mutation proposal behaviour
  strategy: 'fine_search' | 'medium_search' | 'global_search'
           | 'twist_focus' | 'thickness_focus' | 'sweep_focus';

  // Parameter weight genes: per-region exploration weight (sum to 1.0)
  parameter_weights: {
    REGION_PLANFORM_CORE:  number;  // [0.0, 1.0]
    REGION_PLANFORM_SWEEP: number;
    REGION_TWIST:          number;
    REGION_THICKNESS:      number;
    REGION_CAMBER:         number;
    REGION_LE_SHAPING:     number;
    REGION_WINGLET:        number;
  };

  // Step size gene: multiplier on GLOBAL-GEOM-001 step sizes
  step_size_multiplier: number;     // [0.5, 3.0]

  // Patience gene: consecutive fails before strategy escalation
  patience: number;                 // integer [3, 30]

  // Topology gene: probability of allowing topology mutation per exp
  topology_probability: number;     // [0.0, 0.15]

  // Mesh level gene: default mesh level for this genome
  mesh_level: 'coarse' | 'medium' | 'fine';

  // Metadata (not evolved; set by GEP system)
  generation: number;
  fitness: number;
}

Initial genome (generation 0, set at Stage 0):
  strategy:              'fine_search'
  parameter_weights:     uniform (1/7 each, normalised to 1.0)
  step_size_multiplier:  1.0
  patience:              5
  topology_probability:  0.0
  mesh_level:            value of MESH_DEFAULT_LEVEL env var

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-8-001: Pause loop and notify
  (a) Set gep_in_progress = true in ExperimentRun (DB + Redis flag).
  (b) Drain the core queue (wait for in-flight experiment to finish).
  (c) Emit GEP_EVOLUTION_START with run_id, experiment_n, current genome.
  (d) Notify operator via configured notification channel
      (NOTIFICATION_WEBHOOK_URL or NOTIFICATION_EMAIL) with:
        run_id, experiment_n, current best M, current genome summary.

FR-8-002: Serialise current genome before evolution
  Write current genome to:
    .aeroloop/genome/current_genome.json
  Validate against GeneticGenome Zod schema.
  On schema validation failure: abort GEP, retain current genome,
  emit GEP_EVOLUTION_FAILED, resume loop unchanged.

FR-8-003: Evaluate fitness of all candidate genomes (HISTORY-ONLY)
  Generate an initial population of 16 candidate genomes:
    - Candidate 0: the current genome (unchanged)
    - Candidates 1–15: random perturbations of current genome
      (single-gene mutations applied to random genes)
  For each candidate, compute fitness from ExperimentRegistry
  using ALL experiments in this run (including other swarm nodes):

    improvement_rate = count(KEPT) / total_experiments
                       over the experiments that used a genome
                       with the same strategy and step_size_multiplier
                       (tolerance: ±0.2 on step_size_multiplier)

    fitness = 0.6 × improvement_rate
            + 0.3 × mean_delta_M_on_kept (normalised)
            + 0.1 × (1 - timeout_rate)

  If insufficient history for a candidate (< 10 matching experiments),
  use the surrogate model prediction (if available) or assign fitness
  = 0.0 with a note that it is unvalidated.

  No new CFD solves are created during fitness evaluation.

  Write per-candidate fitness scores to:
    .aeroloop/genome/gep_eval_gen{N}.jsonl
  Write checkpoint: { "phase": "EVALUATED", ... }

FR-8-004: Selection — elitism + crossover + mutation + random
  Sort all 16 candidates by fitness descending.

  New generation of 16 genomes:
    Slots  0–1   (2):  Elite survivors — top-2 by fitness, unchanged.
    Slots  2–5   (4):  Crossover offspring — 2 pairs from top-6 fitness;
                       each pair produces 2 children via uniform crossover.
    Slots  6–13  (8):  Single-gene mutants — apply one random gene
                       mutation to 8 randomly selected candidates
                       from the full population (including elites).
    Slots 14–15  (2):  Random new genomes — fully random within bounds;
                       ensures exploration never fully stops.

  Total: 2 + 4 + 8 + 2 = 16. ✓

  Write checkpoint: { "phase": "SELECTED", ... }

FR-8-005: Crossover (uniform crossover on numeric genes)
  For each gene in the genome:
    With probability 0.5, child inherits from parent A; else parent B.
  Exception: strategy gene always inherits from the fitter parent.
  Exception: parameter_weights are normalised to sum to 1.0 after crossover.

FR-8-006: Single-gene mutation bounds
  Each gene has bounded mutation ranges:
    strategy:              sample uniformly from all 6 strategy values
    parameter_weights:     add Gaussian noise (σ=0.05) to one weight,
                           then renormalise all weights to sum to 1.0
    step_size_multiplier:  add Gaussian noise (σ=0.2), clamp to [0.5, 3.0]
    patience:              ± uniform integer in [1, 5], clamp to [3, 30]
    topology_probability:  add Gaussian noise (σ=0.02), clamp to [0.0, 0.15]
    mesh_level:            sample uniformly from ['coarse','medium','fine']

FR-8-007: Select winning genome
  The genome with the highest fitness in the new generation of 16
  is selected as the active genome for the next loop phase.
  If the winner has topology_probability > 0.05 and the previous
  genome had topology_probability == 0.0:
    Emit TOPOLOGY_ENABLED warning event and log.

FR-8-008: Persist evolved genome
  (a) Write winning genome to .aeroloop/genome/current_genome.json
  (b) Write full generation snapshot to:
        .aeroloop/genome/generation_{N}.json
      Contains: winning genome, all 16 candidates with fitness scores.
  (c) Insert row into genomes table (DB):
        generation, evolved_at_experiment_n, all genome fields,
        fitness_before (current genome fitness), fitness_after (winner),
        population_snapshot (all 16 candidates), git_commit_hash.
  (d) git add .aeroloop/genome/
  (e) git commit -m "chore(gep): generation {N} evolved at exp {experiment_n}"
      (matches RULE-061 format)
  (f) Record git_commit_hash in genomes DB row.
  (g) Update ExperimentRun.current_genome and ExperimentRun.gep_generation.
  (h) Write checkpoint: { "phase": "WRITTEN", ... }
  (i) Set gep_in_progress = false.

FR-8-009: Resume loop
  Clear GEP queue lock.
  Emit GEP_EVOLUTION_COMPLETE with generation, fitness_before,
  fitness_after, strategy_before, strategy_after, genome diff.
  Dispatch next experiment (experiment_n + 1) to core queue.

FR-8-010: Topology macro-mutation logging
  Any experiment executed while topology_probability > 0 that
  results in a topology change MUST:
    - Set topology_mutation = true in experiment record
    - Generate a topology wiki article entry (FR-6-005)
    - Be flagged in the genome generation file as a topology event

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-8-001: Trigger, pause and notification
  [ ] GEP fires exactly when experiment_n % GEP_TRIGGER_INTERVAL === 0
  [ ] Core queue does not dispatch experiment N+1 until GEP_EVOLUTION_COMPLETE
  [ ] GEP_EVOLUTION_START emitted before any genome operation
  [ ] Operator notification sent (webhook mock called once) with
      run_id, experiment_n, best_M, and current genome summary
  [ ] If NOTIFICATION_WEBHOOK_URL not set, notification silently skipped
      (no error thrown)

AC-8-002: Genome schema validation
  [ ] A genome file missing the patience field fails Zod validation;
      GEP aborts; current genome retained; GEP_EVOLUTION_FAILED emitted
  [ ] A genome with topology_probability = 0.20 (> 0.15 max) fails
      Zod validation; same abort behaviour

AC-8-003: Population size and composition
  [ ] Initial population before selection contains exactly 16 candidates
  [ ] Candidate 0 is always the current genome (fitness computed from
      history; not a perturbation)
  [ ] After selection, new generation contains exactly 16 genomes:
      2 elites + 4 crossover + 8 single-gene mutants + 2 random

AC-8-004: Elite preservation
  [ ] Top-2 candidates by fitness survive into new generation unchanged
      (gene-for-gene identical; unit test comparing genome objects)
  [ ] Elites are not subjected to crossover or mutation operators

AC-8-005: Fitness evaluation (history-only, no CFD)
  [ ] No BullMQ core queue jobs created during GEP evaluation
      (verified by monitoring queue depth before and after GEP)
  [ ] Fitness formula produces values in [0, 1] for all candidates
      (unit test with fixture experiment history)
  [ ] Candidate with < 10 matching experiments assigned fitness = 0.0
      and flagged as "unvalidated" in gep_eval_gen{N}.jsonl

AC-8-006: Crossover correctness
  [ ] Strategy gene in child always matches the fitter parent (unit test
      with controlled fitness values)
  [ ] parameter_weights in child sum to 1.0 ± 0.001 after crossover
      (unit test; 1000 crossover iterations, all checked)
  [ ] Uniform crossover: each numeric gene has ~50% chance from each
      parent (statistical test over 1000 crossover pairs; p > 0.01)

AC-8-007: Single-gene mutation bounds
  [ ] step_size_multiplier always in [0.5, 3.0] after mutation
      (unit test; 10,000 mutation iterations)
  [ ] patience always in [3, 30] after mutation
  [ ] topology_probability always in [0.0, 0.15] after mutation
  [ ] parameter_weights always sum to 1.0 ± 0.001 after mutation

AC-8-008: Persistence and git integrity
  [ ] .aeroloop/genome/current_genome.json updated after evolution
  [ ] .aeroloop/genome/generation_{N}.json written with all 16
      candidate genomes and their fitness scores
  [ ] genomes table row inserted with correct generation number and
      git_commit_hash matching the actual git commit
  [ ] Git commit message matches RULE-061 format exactly
  [ ] ExperimentRun.gep_generation incremented by 1 in DB

AC-8-009: Topology gate
  [ ] TOPOLOGY_ENABLED event emitted when winner topology_probability
      increases from 0.0 to > 0.05 (boundary test)
  [ ] Event NOT emitted when topology_probability stays 0.0
  [ ] topology_mutation = true set in experiment record for any
      experiment run under topology_probability > 0 that changes
      a topology parameter

AC-8-010: Resume after GEP
  [ ] After GEP_EVOLUTION_COMPLETE, experiment N+1 dispatched within 5 s
  [ ] gep_in_progress = false in DB after completion
  [ ] On cold restart with gep_in_progress = true and checkpoint
      phase = "EVALUATED": GEP re-runs from EVALUATED checkpoint
      (selection and beyond); does not re-run evaluation
  [ ] On cold restart with checkpoint phase = "WRITTEN": GEP is
      considered complete; loop resumes immediately

AC-8-011: Failure recovery
  [ ] Any unhandled exception in GEP: current genome preserved unchanged,
      GEP_EVOLUTION_FAILED emitted, gep_in_progress = false,
      loop resumes with prior genome
  [ ] GEP failure does not halt the loop indefinitely
      (loop resumes within 60 s of failure)


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/stage-9/requirements.md
STAGE 9 — SWARM COORDINATION
═══════════════════════════════════════════════════════════════════════

Purpose: enable N ≥ 2 AeroLoop nodes to search the design space
concurrently without duplicating effort, without race conditions on
wing.geo, and with shared learning via a central knowledge graph and
ExperimentRegistry. Each node is fully autonomous; the swarm layer
is coordination-only and MUST NOT block a node's core loop for more
than 5 seconds.

Constitutional constraints that apply here:
  RULE-001: each node only writes ITS OWN wing.geo
  RULE-006: region claims via atomic Redis SETNX before write
  RULE-003: KEEP/REVERT applies per-node to that node's wing.geo

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SWARM DESIGN REGIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The design space is partitioned into 8 named regions. Each region
maps to a subset of GLOBAL-GEOM-001 parameters.

Region name              Parameters included
──────────────────────────────────────────────────────────────────────
REGION_PLANFORM_CORE     span_m, aspect_ratio, taper_ratio,
                         root_chord_m, tip_chord_m, mac_m
REGION_PLANFORM_SWEEP    le_sweep_inboard_deg, le_sweep_outboard_deg,
                         te_sweep_deg, dihedral_deg
REGION_TWIST             twist_root_deg, twist_25_deg, twist_50_deg,
                         twist_75_deg, twist_tip_deg
REGION_THICKNESS         tc_root, tc_25, tc_50, tc_75, tc_tip
REGION_CAMBER            camber_root, camber_25, camber_50,
                         camber_75, camber_tip
REGION_LE_SHAPING        le_radius_root, le_radius_50, le_radius_tip
REGION_WINGLET           all winglet_* parameters
REGION_GLOBAL            any parameter (no exclusivity; fallback only)

A node that holds a region claim MUST only propose mutations for
parameters in that region's parameter list. Proposals outside the
claimed region are rejected by the swarm guard in Stage 1 (FR-1-008).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTIONAL REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FR-9-001: Node identity and persistence
  Each node has a stable node_id (UUID v4) generated on first start
  and persisted to the file at NODE_ID_FILE (.aeroloop/node_id).
  On subsequent starts, the node reads and reuses this file.
  The file is gitignored.

FR-9-002: Node registration at startup
  Call POST /swarm/nodes with node_id, run_id, hostname, and
  capabilities (core_count, memory_gb, gpu_available).
  If coordinator unreachable within SWARM_API_TIMEOUT_MS:
    Enter OFFLINE_MODE (FR-9-009).

FR-9-003: Heartbeat (every SWARM_HEARTBEAT_S seconds, default 30)
  Call PATCH /swarm/nodes/{node_id} with:
    status, experiment_n, local_best_M, current_region.
  The response includes global_best_M. Update local M_best reference
  if global_best_M > local ExperimentRun.best_M.
  Renew the Redis region claim TTL on every successful heartbeat.
  Three consecutive heartbeat failures → NODE_STALE declared by
  coordinator; grace period = 10 minutes before claims released.

FR-9-004: Region claim protocol
  Before writing wing.geo (called from FR-1-008):
    (a) Determine preferred region from genome strategy:
          fine_search / medium_search  → REGION_PLANFORM_CORE first
          twist_focus                  → REGION_TWIST first
          thickness_focus              → REGION_THICKNESS first
          sweep_focus                  → REGION_PLANFORM_SWEEP first
          global_search                → REGION_GLOBAL
          no match                     → round-robin unclaimed regions
    (b) Attempt Redis SETNX:
          Key:   aeroloop:{run_id}:region:{region_name}:owner
          Value: {node_id}
          TTL:   SWARM_REGION_TTL_S (default 600 s)
    (c) If SETNX returns 0 (another node owns it): try next region
        in preference order. Attempt all 8 regions before giving up.
    (d) If all regions claimed: wait 60 s polling (5 s intervals);
        then claim REGION_GLOBAL as fallback (no exclusivity; log WARNING).
    (e) A node holds at most 2 region claims: one active, one
        pre-claimed for the next batch.

FR-9-005: Region validation (swarm guard in Stage 1)
  After a mutation proposal is generated (FR-1-002), before bounds
  check: verify the proposed parameter_id is in the parameter list
  for the node's currently claimed region.
  If not: reject the proposal; re-propose within the claimed region.
  If REGION_GLOBAL is claimed: any parameter is valid.

FR-9-006: Shared experiment registry push
  After every KEEP or REVERT decision (Stage 5 completes):
    Call POST /experiments (as in Stage 5, normal path).
  In OFFLINE_MODE: accumulate records locally in
    .aeroloop/offline_queue.jsonl
  On reconnect: flush offline_queue.jsonl to POST /experiments in
  chronological order. Check for 409 conflicts (duplicate experiment_n
  records from another node); log and skip duplicates.

FR-9-007: Shared knowledge graph pull
  GET /knowledge-graph/summary is called before every mutation proposal
  (FR-1-001(d)). This already pulls from the shared KG, so in swarm
  mode, each node benefits from all other nodes' learning automatically.

FR-9-008: Global best M synchronisation
  Global best M is stored at:
    Redis key: aeroloop:{run_id}:global_best_M
  Updated by KeepRevertAgent (FR-5-002) via Redis GETSET.
  Read by SwarmSyncAgent on every heartbeat response.
  A node that receives a higher global_best_M than its local best
  updates its local M_best reference. This means future KEEP decisions
  are made against the run-wide best, not just the node's local best.

FR-9-009: OFFLINE_MODE
  When coordinator is unreachable (POST/PATCH timeout > SWARM_API_TIMEOUT_MS):
    (a) Emit SWARM_OFFLINE_MODE event
    (b) Node continues its loop independently
    (c) Experiment records accumulated in .aeroloop/offline_queue.jsonl
    (d) Region claims: use last successfully held claim for up to
        SWARM_REGION_TTL_S seconds; after that, all parameters valid
    (e) On reconnect (coordinator reachable on next heartbeat attempt):
          re-register, re-claim region, flush offline_queue.jsonl
          Emit SWARM_RECONNECTED

FR-9-010: Region conflict resolution
  If two nodes acquire the same region claim simultaneously (due to
  Redis partition or network split):
    Detected on heartbeat: coordinator compares owner from Redis
    vs DB swarm_nodes.current_region for each node.
    Resolution: the node with the lexicographically smaller node_id
    retains the claim; the other releases and re-claims a free region.
    Emit REGION_CONFLICT_RESOLVED on the losing node.

FR-9-011: Clean shutdown
  On SIGTERM or operator DELETE /swarm/nodes/{node_id}:
    (a) Wait for current experiment to complete (not mid-stage)
    (b) Release all Redis region claims (DEL)
    (c) Call DELETE /swarm/nodes/{node_id}
    (d) Flush offline_queue.jsonl if non-empty
    (e) Exit cleanly

FR-9-012: Swarm SSE stream and dashboard feed
  The coordinator API emits all swarm events to the SSE stream at
  GET /swarm/stream. The dashboard (apps/dashboard) consumes this
  stream and renders:
    - Active nodes grid (id, hostname, status, region, local best M,
      experiment rate in experiments/hour rolling 1h)
    - Global best M over time (line chart, all nodes combined)
    - Region heatmap (which node owns which region, claim age as colour)
    - Global experiment log (last 50 events, all nodes, reverse chron)
  Dashboard is READ-ONLY. No operator commands from the dashboard;
  use apps/cli for all control operations.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACCEPTANCE CRITERIA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-9-001: Node identity
  [ ] node_id persisted to NODE_ID_FILE on first start
  [ ] Same node_id reused on restart (file read; not regenerated)
  [ ] Different nodes on different machines have different node_ids

AC-9-002: Registration and heartbeat
  [ ] Node registered within 3 s of first loop iteration
  [ ] Heartbeat fires every 30 ± 2 s (verified over 5 minute window)
  [ ] Node marked STALE in DB within 95 s of last heartbeat
      (3 × 30 + 5 s tolerance)
  [ ] Clean shutdown: DELETE /swarm/nodes called within 2 s of SIGTERM;
      region claims released; swarm_nodes.deregistered_at set

AC-9-003: Region claim non-collision (IT-006)
  [ ] With 4 nodes, at most 1 node holds each named region at any time
      (Redis SETNX guarantee verified by polling Redis at 100 ms intervals
      over 500 experiments)
  [ ] No wing.geo write conflicts in git log across 4-node run
  [ ] With 4 nodes, at least 3 distinct regions claimed simultaneously
      within 5 experiments of startup

AC-9-004: Region validation
  [ ] A proposal for taper_ratio (REGION_PLANFORM_CORE) while holding
      REGION_TWIST is rejected and re-proposed (unit test)
  [ ] A proposal for any parameter while holding REGION_GLOBAL is
      accepted (unit test)

AC-9-005: Shared registry and knowledge sync
  [ ] A KEPT experiment on node-A is visible to node-B's KG summary
      within 2 heartbeat cycles (≤ 60 s) in integration test
  [ ] Global best M updated in Redis within 5 s of a cross-node
      best-improvement event
  [ ] OFFLINE_MODE accumulation: node offline for 50 experiments,
      reconnects, all 50 records in shared DB in order,
      zero 409 conflicts (IT-006 extension)

AC-9-006: OFFLINE_MODE
  [ ] Coordinator restart does not halt any node's loop
  [ ] SWARM_OFFLINE_MODE event emitted on first heartbeat failure
  [ ] SWARM_RECONNECTED event emitted when coordinator comes back
  [ ] offline_queue.jsonl flushed in chronological order on reconnect

AC-9-007: Conflict resolution
  [ ] Injected dual-SETNX race (simulated): losing node releases claim
      within 2 heartbeat cycles and re-claims a different region
  [ ] REGION_CONFLICT_RESOLVED emitted on losing node

AC-9-008: Dashboard
  [ ] SSE events arrive at dashboard within 500 ms of occurrence
  [ ] Dashboard renders with no stale data older than 35 s
  [ ] Dashboard shows correct node count, region assignment, and
      global best M matching the DB value (verified by API cross-check)
  [ ] No operator command can be issued from dashboard UI


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/infra.md
GLOBAL-INFRA-001 — INFRASTRUCTURE & CI/CD
═══════════════════════════════════════════════════════════════════════

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MONOREPO DIRECTORY STRUCTURE (complete)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

aeroloop/
│
├── packages/                          Shared libraries (no runnable apps)
│   │
│   ├── types/                         Zod schemas + TypeScript types
│   │   ├── src/
│   │   │   ├── experiment.ts          ExperimentRun, ExperimentRecord
│   │   │   ├── genome.ts              GeneticGenome, GeneticGene
│   │   │   ├── knowledge-graph.ts     Sensitivity, Interaction,
│   │   │   │                          DetectImpactRequest/Response
│   │   │   ├── geometry.ts            ParameterRegistry, BoundsResult
│   │   │   ├── messages.ts            LoopMessage, MessageType, Stage
│   │   │   ├── swarm.ts               SwarmNode, RegionClaim
│   │   │   ├── env.ts                 EnvSchema (GLOBAL-ENV-001)
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── db/                            Database client + Drizzle ORM schema
│   │   ├── src/
│   │   │   ├── schema/
│   │   │   │   ├── runs.ts
│   │   │   │   ├── experiments.ts
│   │   │   │   ├── genomes.ts
│   │   │   │   ├── knowledge-graph.ts
│   │   │   │   └── wiki.ts
│   │   │   ├── client.ts
│   │   │   ├── migrations/            Drizzle migration files
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── redis/                         Redis client + key helpers
│   │   ├── src/
│   │   │   ├── client.ts
│   │   │   ├── keys.ts                All Redis key templates (single source)
│   │   │   ├── swarm-claims.ts        SETNX claim acquire/renew/release
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── s3/                            S3 client + upload/download helpers
│   │   ├── src/
│   │   │   ├── client.ts
│   │   │   ├── mesh.ts                Mesh upload/download + SHA256 sidecar
│   │   │   ├── results.ts             Solver output upload/download + sidecar
│   │   │   ├── geometry.ts            wing.geo snapshot upload
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── geometry/                      Geometry utilities
│   │   ├── src/
│   │   │   ├── parameter-registry.ts  Full registry + hard bounds constants
│   │   │   ├── bounds-checker.ts      Hard bounds + cross-param constraints
│   │   │   ├── wing-geo-parser.ts     Read/write wing.geo variable values
│   │   │   ├── sha256.ts              File SHA256 computation utility
│   │   │   └── index.ts
│   │   ├── tests/
│   │   │   ├── unit/
│   │   │   │   ├── bounds-checker.test.ts
│   │   │   │   ├── wing-geo-parser.test.ts
│   │   │   │   └── parameter-registry.test.ts
│   │   │   └── integration/
│   │   └── package.json
│   │
│   ├── meshing/                       GMSH pipeline wrapper
│   │   ├── src/
│   │   │   ├── auto-mesh.ts           Subprocess caller for auto_mesh.py
│   │   │   ├── quality-checker.ts     GLOBAL-MESH-001 gate implementation
│   │   │   ├── yplus-calculator.ts    First-cell height computation
│   │   │   ├── templates/             GMSH .geo templates per level
│   │   │   │   ├── coarse.geo.tmpl
│   │   │   │   ├── medium.geo.tmpl
│   │   │   │   └── fine.geo.tmpl
│   │   │   └── index.ts
│   │   ├── python/
│   │   │   └── auto_mesh.py           GMSH Python API meshing script
│   │   ├── fixtures/                  Reference meshes for tests
│   │   └── package.json
│   │
│   ├── solver/                        SU2 runner + kill timer
│   │   ├── src/
│   │   │   ├── su2-runner.ts          Real SU2Runner (mpirun + dual timeout)
│   │   │   ├── mock-su2-runner.ts     MockSU2Runner (returns fixtures)
│   │   │   ├── convergence-checker.ts Residual + force stabilisation check
│   │   │   ├── config-generator.ts    SU2 cfg template rendering
│   │   │   ├── config-templates/      SU2 .cfg templates
│   │   │   │   ├── rans_sa.cfg.tmpl
│   │   │   │   └── rans_kwsst.cfg.tmpl
│   │   │   └── index.ts
│   │   ├── fixtures/                  Pre-recorded surface_flow.csv,
│   │   │                              history.csv for tests
│   │   └── package.json
│   │
│   ├── extraction/                    CFD results extraction
│   │   ├── src/
│   │   │   ├── polar-extractor.ts     Parse surface_flow.csv → CL, CD, CMy
│   │   │   ├── wave-drag.ts           Far-field + fallback Cd_wave
│   │   │   ├── buffet-detection.ts    Alpha sweep slope analysis
│   │   │   ├── metric-computer.ts     GLOBAL-METRIC-001 formula
│   │   │   ├── sanity-checker.ts      GLOBAL-METRIC-001 sanity bounds
│   │   │   └── index.ts
│   │   ├── fixtures/                  Reference surface_flow.csv files
│   │   └── package.json
│   │
│   ├── git-ops/                       Git keep/revert primitives
│   │   ├── src/
│   │   │   ├── keeper.ts              git add + commit with RULE-060 format
│   │   │   ├── reverter.ts            git checkout geometry/wing.geo
│   │   │   ├── clean-check.ts         git status --porcelain dirty check
│   │   │   ├── commit-validator.ts    RULE-060/061 regex validation
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── knowledge-graph/               KG update + detect-impact engine
│   │   ├── src/
│   │   │   ├── updater.ts             FR-7-001, FR-7-002 implementation
│   │   │   ├── impact-detector.ts     FR-7-006 detect-impact logic
│   │   │   ├── interaction-discovery.ts  FR-7-005 Mann–Whitney U test
│   │   │   ├── dead-zone-tracker.ts   FR-7-003 dead zone logic
│   │   │   ├── hot-zone-tracker.ts    FR-7-004 hot zone logic
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── surrogate/                     Gaussian Process surrogate model
│   │   ├── src/
│   │   │   ├── trainer.ts             GPR training via Python subprocess
│   │   │   ├── predictor.ts           GPR prediction via Python subprocess
│   │   │   └── index.ts
│   │   ├── python/
│   │   │   ├── train_gp.py            scikit-learn GP training script
│   │   │   └── predict_gp.py          scikit-learn GP prediction script
│   │   └── package.json
│   │
│   ├── wiki/                          Wiki compiler
│   │   ├── src/
│   │   │   ├── compiler.ts            Orchestrates all article types
│   │   │   ├── sensitivity-article.ts FR-6-002 implementation
│   │   │   ├── batch-summary-article.ts FR-6-003 implementation
│   │   │   ├── interaction-article.ts FR-6-004 implementation
│   │   │   ├── topology-article.ts    FR-6-005 implementation
│   │   │   ├── kg-summary-writer.ts   FR-6-006 implementation
│   │   │   ├── vagueness-guard.ts     Rejects articles with vague phrases
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   └── gep/                           GEP genome evolver
│       ├── src/
│       │   ├── evolver.ts             Main GEP orchestration (FR-8-001–009)
│       │   ├── fitness.ts             FR-8-003 fitness computation
│       │   ├── operators.ts           FR-8-005/006 crossover + mutation
│       │   ├── selector.ts            FR-8-004 selection (2+4+8+2)
│       │   ├── genome-schema.ts       GeneticGenome Zod schema
│       │   ├── checkpoint.ts          Read/write gep_checkpoint.json
│       │   └── index.ts
│       └── package.json
│
├── apps/
│   │
│   ├── loop/                          Main loop orchestrator (Bun process)
│   │   ├── src/
│   │   │   ├── orchestrator.ts        Stage state machine (RULE-007)
│   │   │   ├── agents/
│   │   │   │   ├── mutation-agent.ts  GeometryMutationAgent
│   │   │   │   ├── mesh-agent.ts      MeshAgent
│   │   │   │   ├── mesh-quality-agent.ts  MeshQualityAgent
│   │   │   │   ├── solver-agent.ts    SolverAgent
│   │   │   │   ├── extraction-agent.ts  ExtractionAgent
│   │   │   │   ├── metric-agent.ts    MetricAgent
│   │   │   │   ├── keep-revert-agent.ts  KeepRevertAgent
│   │   │   │   ├── wiki-agent.ts      WikiCompilerAgent
│   │   │   │   ├── kg-agent.ts        KnowledgeGraphAgent
│   │   │   │   ├── surrogate-agent.ts SurrogateAgent
│   │   │   │   ├── gep-agent.ts       GEPOrchestrator
│   │   │   │   └── swarm-agent.ts     SwarmSyncAgent
│   │   │   ├── queue/
│   │   │   │   └── bullmq-setup.ts    Queue definitions + job options
│   │   │   ├── dirty-check.ts         Pre-experiment dirty tree guard
│   │   │   └── main.ts                Entry point; env validation; startup
│   │   └── package.json
│   │
│   ├── api/                           REST API + SSE server (Hono, Bun)
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
│   │   │   │   ├── auth.ts            JWT bearer validation
│   │   │   │   └── rate-limit.ts      1000 req/min per node_id
│   │   │   ├── sse/
│   │   │   │   └── swarm-stream.ts    SSE event emitter for /swarm/stream
│   │   │   ├── validation/
│   │   │   │   └── schemas.ts         Zod schemas for all request bodies
│   │   │   └── main.ts
│   │   └── package.json
│   │
│   ├── dashboard/                     Read-only swarm dashboard (React + Vite)
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── NodeGrid.tsx
│   │   │   │   ├── MetricChart.tsx    Global best M over time (line chart)
│   │   │   │   ├── RegionHeatmap.tsx  Region ownership + claim age
│   │   │   │   ├── ExperimentLog.tsx  Last 50 events, reverse chronological
│   │   │   │   └── GlobalBestBanner.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useSwarmStream.ts  EventSource consumer + state
│   │   │   ├── pages/
│   │   │   │   ├── Overview.tsx       Main dashboard page
│   │   │   │   ├── RunDetail.tsx      Single run stats + experiment table
│   │   │   │   └── ExperimentDetail.tsx  Single experiment all fields
│   │   │   └── main.tsx
│   │   └── package.json
│   │
│   └── cli/                           Operator CLI (Bun CLI)
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
│   │   ├── Dockerfile.loop            Bun loop process image
│   │   ├── Dockerfile.api             Bun API server image
│   │   ├── Dockerfile.su2             SU2 + OpenMPI pre-built image
│   │   ├── Dockerfile.gmsh            GMSH + Python 3 image
│   │   └── docker-compose.yml         Full local dev stack
│   │
│   ├── k8s/
│   │   ├── namespace.yaml
│   │   ├── api-deployment.yaml
│   │   ├── loop-statefulset.yaml      StatefulSet (stable node IDs)
│   │   ├── redis-deployment.yaml
│   │   ├── postgres-statefulset.yaml
│   │   ├── minio-deployment.yaml
│   │   ├── hpa-loop.yaml              HorizontalPodAutoscaler for loop nodes
│   │   └── ingress.yaml
│   │
│   ├── terraform/                     Optional cloud infra (AWS / GCP)
│   │   ├── main.tf
│   │   ├── rds.tf
│   │   ├── elasticache.tf
│   │   ├── s3.tf
│   │   └── variables.tf
│   │
│   └── scripts/
│       ├── bootstrap.sh               First-time dev environment setup
│       ├── seed-db.sh                 Seed parameter registry + pre-seeded KG
│       └── health-check.sh            Verify all services reachable
│
├── geometry/
│   └── wing.geo                       THE ONLY MUTABLE FILE IN THE LOOP
│                                      (RULE-001; modified only by
│                                       GeometryMutationAgent)
│
├── .aeroloop/                         Runtime state directory
│   │                                  (partially gitignored — see below)
│   ├── run_id                         gitignored
│   ├── node_id                        gitignored
│   ├── best_metric.json               gitignored
│   ├── decision_log.jsonl             gitignored
│   ├── offline_queue.jsonl            gitignored
│   ├── mesh_quality_exp_*.json        gitignored
│   ├── surrogate/                     gitignored
│   │   └── model.pkl
│   ├── knowledge_graph_summary.md     committed (updated by Stage 6)
│   ├── wiki/                          committed (managed by Stage 6)
│   └── genome/                        committed (managed by Stage 8)
│       ├── current_genome.json
│       ├── generation_*.json
│       └── gep_checkpoint.json        gitignored
│
├── hooks/
│   ├── pre-commit                     RULE-001 enforcement
│   └── commit-msg                     RULE-060/061 enforcement
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── integration.yml
│       └── release.yml
│
├── CLAUDE.md                          → see Part 1 of this spec
├── AGENTS.md                          → see Part 1 of this spec
├── program.md                         → see Part 1 of this spec
├── Aeroloopspec.md                    This document
├── turbo.json                         Turborepo pipeline config
├── pnpm-workspace.yaml
└── package.json

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DOCKER COMPOSE (local dev stack)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Services:

Service          Image                      Port(s)   Notes
──────────────────────────────────────────────────────────────────────
postgres         postgres:15-alpine         5432      Persistent volume
redis            redis:7-alpine             6379      Persistent AOF
minio            minio/minio                9000      S3-compat; console 9001
api              aeroloop/api:dev           4000      Hot-reload via Bun --watch
loop             aeroloop/loop:dev          –         Single node; dev mode
dashboard        aeroloop/dashboard:dev     3000      Vite HMR
bullboard        ghcr.io/felixmosh/bull-board 3001    Queue visibility UI
su2              aeroloop/su2:latest        –         SU2 binary only; no port
gmsh             aeroloop/gmsh:latest       –         GMSH + Python; no port

All services read configuration from .env (template in GLOBAL-ENV-001).

Health check order (bootstrap.sh enforces startup order):
  postgres → redis → minio → api → loop

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CI PIPELINE (.github/workflows/ci.yml)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Triggers: push to any branch; pull_request targeting main.
All jobs must pass for merge to main (branch protection rule).

Job 1: typecheck
  - uses: actions/setup-node@v4 (node 20) + Bun setup
  - run: pnpm install --frozen-lockfile
  - run: pnpm turbo typecheck
  TypeScript strict mode; zero type errors.

Job 2: lint
  - run: pnpm turbo lint
  ESLint with @typescript-eslint + eslint-plugin-drizzle.
  Zero warnings policy: all warnings treated as errors in CI.
  no-console rule enforced (use pino logger).

Job 3: unit-test
  - run: pnpm turbo test
  Coverage thresholds enforced (GLOBAL-TEST-001).
  Build fails if any package is below its threshold.

Job 4: integration-test
  Services (via docker-compose test profile):
    postgres, redis, minio (MinIO testcontainer or compose service)
  - run: pnpm turbo test:integration
  Must pass IT-001 through IT-005.
  Real GMSH; MockSU2Runner; real PostgreSQL; real Redis.
  Timeout: 20 minutes.

Job 5: build
  - run: pnpm turbo build
  All packages and apps build without error.
  Verifies no circular dependencies via Turborepo dependency graph.

Job 6: docker-build
  Builds all 4 Dockerfiles (loop, api, su2, gmsh).
  Does NOT push images (push only on release tag).
  Verifies Docker layer caching is effective (< 3 min per image on cache hit).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RELEASE PIPELINE (.github/workflows/release.yml)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Trigger: push of tag matching v*.*.* (semver).

Steps:
  1. Run full CI pipeline (all 6 jobs above)
  2. Build and push Docker images to registry with tag = git tag
  3. Generate changelog from conventional commits since last tag
  4. Create GitHub Release with changelog and Docker image references
  5. Deploy to staging k8s cluster (automatic on tag push)
  6. Deploy to production k8s cluster (requires manual approval in
     GitHub environment protection rules)

Image tags pushed:
  aeroloop/loop:{version}   aeroloop/loop:latest
  aeroloop/api:{version}    aeroloop/api:latest
  aeroloop/su2:{version}    (pinned; only rebuilt when SU2 version changes)
  aeroloop/gmsh:{version}   (pinned; only rebuilt when GMSH version changes)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TURBOREPO PIPELINE (turbo.json)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "typecheck": {
      "dependsOn": ["^build"]
    },
    "lint": {},
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "test:integration": {
      "dependsOn": ["^build"],
      "cache": false
    }
  }
}

Package build order (enforced by RULE-012 dependency chain):
  types → db → redis → s3 → geometry → meshing → solver →
  extraction → git-ops → knowledge-graph → surrogate → gep → wiki
  → [apps: loop, api, dashboard, cli]


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/operator/runbook.md
OPERATOR-001 — OPERATOR RUNBOOK
═══════════════════════════════════════════════════════════════════════

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FIRST-TIME SETUP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# 1. Clone repo and install dependencies
git clone https://github.com/knarayanareddy/Aeroloop.git
cd Aeroloop
pnpm install

# 2. Copy and configure environment
cp .env.example .env
# Edit .env: set SU2_BINARY, GMSH_PYTHON, DATABASE_URL,
#            REDIS_URL, S3_*, JWT_SECRET

# 3. Install git hooks
cp hooks/pre-commit .git/hooks/pre-commit
cp hooks/commit-msg .git/hooks/commit-msg
chmod +x .git/hooks/pre-commit .git/hooks/commit-msg

# 4. Start local infrastructure
docker compose up -d postgres redis minio
./infra/scripts/bootstrap.sh

# 5. Run database migrations
pnpm --filter db drizzle-kit migrate

# 6. Verify environment
pnpm cli status --check-env
# Expected: all variables valid; DB/Redis/S3 green

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STARTING A SINGLE-NODE RUN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# 1. Ensure git working tree is clean
git status
# Must show: nothing to commit, working tree clean

# 2. Start the loop
pnpm cli start \
  --run-label   "naca0012-sweep-001" \
  --mach        0.78 \
  --reynolds    4.5e7 \
  --alpha       2.5 \
  --w-ld        0.5 \
  --w-buffet    0.3 \
  --w-wave      0.2 \
  --kill-timer  480

# 3. Monitor
open http://localhost:3000          # Dashboard
pnpm cli status                     # CLI status summary

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STARTING A SWARM RUN (N NODES)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# On the coordinator node (also runs the API server):
docker compose up -d api
pnpm cli start --swarm \
  --run-label "sweep-swarm-001" \
  --mach 0.78 --reynolds 4.5e7 --alpha 2.5 \
  --w-ld 0.5 --w-buffet 0.3 --w-wave 0.2

# Note the run_id printed on stdout.

# On each worker node (different machines or containers):
export SWARM_ENABLED=true
export NODE_COORDINATOR_URL=http://coordinator-host:4000
pnpm cli start --join-run {run_id}

# Each worker node will register, claim a region, and begin looping.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
COMMON OPERATOR COMMANDS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pause a run (waits for current experiment to finish):
  pnpm cli pause --run-id {run_id} [--node-id {node_id}|--all]

Resume a paused run:
  pnpm cli resume --run-id {run_id} [--node-id {node_id}|--all]

Abort the current experiment immediately:
  pnpm cli abort --run-id {run_id} --node-id {node_id}
  # Kill timer fires; geometry reverted; loop continues.

Check full system status:
  pnpm cli status --run-id {run_id}
  # Output format:
  # Run:              sweep-swarm-001 (RUNNING)
  # Experiments:      144 total | 41 kept (28.5%) | 2 timeouts | 1 mesh fail
  # Best M:           0.8821 @ exp-138 | geometry_sha256: abc123...
  # Rate:             8.2 experiments/hour (rolling 1h)
  # Consecutive fails: 0
  # GEP:              generation 0 | next at exp 500
  # Active nodes:     4 | regions: PLANFORM_CORE, TWIST, THICKNESS, CAMBER
  # Health:           DB ok | Redis ok | S3 ok

Export results:
  pnpm cli export --run-id {run_id} --format csv --output ./results.csv
  pnpm cli export --run-id {run_id} --format json --output ./full_history.json
  pnpm cli export --run-id {run_id} --type kg --output ./kg_summary.json

View experiment detail:
  pnpm cli run-history --run-id {run_id} --status KEPT --limit 20
  pnpm cli run-history --run-id {run_id} --experiment-n 144

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
INCIDENT RECOVERY PROCEDURES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

INCIDENT: LOOP_HALT_DIRTY_TREE
  Cause:  wing.geo (or another file) left in a modified state without
          commit or revert. Usually caused by a crash mid-Stage 5.
  Resolution:
    git status                       # identify dirty files
    git checkout geometry/wing.geo   # restore wing.geo to HEAD
    git status                       # must be clean
    pnpm cli resume --run-id {run_id}

INCIDENT: LOOP_CONSECUTIVE_FAIL (3 consecutive non-KEPT outcomes)
  Cause:  design space stuck; strategy too conservative; dead zone
          declared on all attempted parameters.
  Resolution:
    pnpm cli status --run-id {run_id}     # check which parameters failed
    # Option A: resume (loop will try different parameters via dead-zone avoidance)
    pnpm cli resume --run-id {run_id}
    # Option B: if stuck after resume, wait for GEP evolution to improve strategy
    # Option C: manually abort current GEP trigger interval and force next evolution
    #   (not yet automated; requires direct DB update as last resort)

INCIDENT: SOLVE_TIMEOUT rate > 30% over last 50 experiments
  Cause:  KILL_TIMER_S too low for current geometry complexity;
          or solver diverging quickly on aggressive mutations.
  Resolution:
    # Option A: increase KILL_TIMER_S in .env and restart loop
    # Option B: edit genome strategy via DB to lower step_size_multiplier:
    UPDATE experiment_runs
      SET current_genome = jsonb_set(current_genome,
          '{step_size_multiplier}', '0.5')
      WHERE run_id = '{run_id}';
    pnpm cli resume --run-id {run_id}

INCIDENT: MESH_FAIL rate > 20% over last 50 experiments
  Cause:  mutations too aggressive causing GMSH to fail or produce
          poor quality meshes; or topology change created bad geometry.
  Resolution:
    # Check recent mutation deltas in experiment history:
    pnpm cli run-history --run-id {run_id} --status MESH_FAIL --limit 10
    # If topology mutations caused failures:
    UPDATE experiment_runs
      SET current_genome = jsonb_set(current_genome,
          '{topology_probability}', '0.0')
      WHERE run_id = '{run_id}';
    pnpm cli resume --run-id {run_id}

INCIDENT: Coordinator down (swarm mode)
  Cause:  apps/api process or pod crashed.
  Resolution:
    # Nodes automatically enter OFFLINE_MODE.
    docker compose restart api       # or kubectl rollout restart deploy/api
    # Nodes detect reconnect within 30 s and re-register.
    # Verify offline records synced:
    pnpm cli status --run-id {run_id} --check-sync

INCIDENT: Redis data loss (flush or restart without AOF)
  Cause:  Redis restarted without persistence; all claims, locks, and
          global_best_M lost.
  Resolution:
    # All nodes will fail to renew claims; region guard will fail open.
    # Stop all loop nodes:
    pnpm cli pause --run-id {run_id} --node-id ALL
    # Restore global_best_M from DB:
    #   SELECT best_M FROM experiment_runs WHERE run_id = '{run_id}';
    redis-cli SET aeroloop:{run_id}:global_best_M {value}
    # Restart Redis with AOF enabled.
    # Resume nodes one at a time:
    pnpm cli resume --run-id {run_id} --node-id {node_id_1}
    # Wait 35 s for region claims to stabilise, then resume others.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NIGHTLY RUN CHECKLIST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Before leaving a run unattended overnight:
  [ ] pnpm cli status shows RUNNING with 0 consecutive fails
  [ ] NOTIFICATION_WEBHOOK_URL set (to receive GEP and fail alerts)
  [ ] docker stats shows memory within 80% on all nodes
  [ ] S3 bucket has > 10 GB free (medium mesh ~200 MB per experiment)
  [ ] PostgreSQL has > 2 GB free
  [ ] git log shows clean KEEP commits with no merge conflicts
  [ ] .aeroloop/decision_log.jsonl exists and is non-empty
  [ ] Last 10 experiments show healthy mix of KEPT and REVERTED
      (> 70% REVERTED is normal; > 99% REVERTED may indicate a stuck run)


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/open-items.md
GLOBAL-OPEN-001 — OPEN ITEMS & KNOWN GAPS
═══════════════════════════════════════════════════════════════════════

These items are known to be underspecified as of spec v1.0. Each must
be resolved before Phase 5 production hardening (see program.md).
No implementation should silently assume a resolution; the ID must be
cited in a code comment when a placeholder or workaround is used.

ID        Area                Gap description
────────────────────────────────────────────────────────────────────────────
OI-001    Surrogate model     Algorithm specified (GPR) but prediction API
                              endpoint not defined. Resolution: add
                              GET /api/v1/knowledge-graph/surrogate-predict
                              ?parameter_id={id}&proposed_value={val} to
                              GLOBAL-API-001 in next spec revision.

OI-002    Multi-objective     Current metric collapses to scalar M. Pareto
                              front exploration not supported. Resolution
                              (post-v1.0): add pareto_mode flag to run
                              config; store CL/CD/buffet as separate
                              objectives; use NSGA-II as optional GEP
                              strategy gene value.

OI-003    Turbulence model    SU2 config template defaults to SA model.
                              k-ω SST may be more accurate for high-AoA
                              buffet prediction. Resolution: validate both
                              models against reference NACA 0012 polars at
                              Mach 0.78 during Phase 5; make the default
                              data-driven; document in solver_config schema.

OI-004    2D geometry mode    Spec assumes 3D wing.geo. No 2D airfoil
                              optimisation mode defined. Resolution: add
                              geometry_mode: '2D'|'3D' to run config; 2D
                              uses single-section GMSH template and
                              different SU2 boundary conditions.

OI-005    Authentication      JWT auth specified but no user management
                              (registration, roles) defined. Resolution
                              for v1.0: single shared JWT_SECRET via env.
                              Post-v1.0: add users table + role enum
                              (admin | read-only).

OI-006    KG merge strategy   Last-write-wins for swarm KG sensitivity
                              merges may lose nuance when two nodes explore
                              the same parameter simultaneously from
                              different directions. Resolution (post-v1.0):
                              weighted Bayesian update (weight by sample
                              count) instead of last-write-wins.

OI-007    Compute cost        No cost tracking per experiment. Swarm runs
                              have no budget governor. Resolution: add
                              cost_usd field to ExperimentRecord; compute
                              as solve_wall_time_s × NODE_COST_PER_HOUR
                              (env var); add budget_usd_limit to run config
                              with loop halt on breach.

OI-008    wing.geo format     wing.geo GMSH macro syntax, variable naming
                              conventions, and section IDs are assumed but
                              not formally specified. Resolution: publish
                              GLOBAL-GEOM-002 wing.geo format spec before
                              Phase 1 begins (blocking for packages/geometry
                              parser implementation).

OI-009    Solver restart      SU2 restart files (.dat) not yet specified.
                              On timeout or unconverged solve, warm restart
                              from a previous checkpoint could save compute.
                              Resolution: add solver_restart: boolean to
                              genome; implement SU2 restart file management
                              in packages/solver.

OI-010    Dashboard auth      Dashboard has no auth in dev mode. SSE stream
                              uses query-param token (workaround for browser
                              EventSource limitation). Resolution for v1.0:
                              add DASHBOARD_READ_TOKEN env var for staging.
                              Post-v1.0: use WebSocket with proper auth.

OI-011    wing.geo format     GLOBAL-GEOM-002 (wing.geo format spec) is
          (blocking for P1)   referenced in OI-008 and is blocking for
                              packages/geometry implementation. This must be
                              written and merged before Phase 1 kick-off.
                              Owner: assign before Phase 0 milestone gate.


═══════════════════════════════════════════════════════════════════════
VIRTUAL FILE: specs/_global/changelog.md
GLOBAL-CHANGELOG — SPECIFICATION CHANGELOG
═══════════════════════════════════════════════════════════════════════

Version  Date        Author             Changes
──────────────────────────────────────────────────────────────────────
0.1      2025-01-01  knarayanareddy    Initial draft: Stages 0–5 core
                                        pipeline; constitutional rules;
                                        agent registry.

0.2      2025-01-08  knarayanareddy    Added Stage 8 GEP requirements;
                                        AGENTS.md message protocol;
                                        knowledge graph Stage 7 outline.

0.3      2025-01-12  knarayanareddy    Added Stage 9 swarm constitutional
                                        law; Stage 6 wiki requirements;
                                        technology stack declared
                                        (Bun + Hono).

1.0      2025-01-15  knarayanareddy    COMPLETE SPEC — all sections
                                        written and internally consistent:

                                        Constitutional fixes:
                                        - RULE-003: REVERT path changed
                                          from "git revert HEAD" to
                                          "git checkout geometry/wing.geo"
                                          throughout all stages
                                        - RULE-060: single canonical KEEP
                                          commit format; all duplicate/
                                          conflicting formats removed
                                        - RULE-061: GEP + wiki commit
                                          formats formalised
                                        - RULE-012: dependency chain
                                          updated to include surrogate
                                          package

                                        Stack consistency:
                                        - API framework: Hono (Bun) —
                                          "FastAPI/Hono" slash removed
                                          from all locations
                                        - CLI: Bun CLI (Commander.js
                                          reference removed)

                                        Stage 8 GEP consistency:
                                        - Population = 16 declared in
                                          FR-8-004 and all ACs
                                        - Elites = top-2 declared in
                                          FR-8-004 and AC-8-004
                                        - Selection breakdown: 2+4+8+2=16
                                          (was inconsistent; now single
                                          canonical definition)
                                        - AC-8-001 through AC-8-011 added
                                          (previously only AC-8-001 existed)

                                        Data model completeness:
                                        - genomes table added to
                                          GLOBAL-DM-001 (was referenced
                                          in FR-8-008 but absent from schema)
                                        - swarm_nodes table added
                                        - All indexes documented

                                        New sections (all complete):
                                        - Stage 9 swarm (FR + AC)
                                        - GLOBAL-API-001 (all 7 endpoint
                                          groups with full schemas)
                                        - GLOBAL-MSG-001 (complete enum,
                                          47 message types; Redis key schema;
                                          BullMQ queue layout)
                                        - GLOBAL-METRIC-001 (formula,
                                          fallback, wave drag, sanity bounds,
                                          versioning)
                                        - GLOBAL-MESH-001 (quality gates,
                                          level targets, report format)
                                        - GLOBAL-ENV-001 (Zod schema,
                                          .env template)
                                        - GLOBAL-COMMIT-001 (hook code,
                                          S3 path conventions)
                                        - GLOBAL-LOG-001 (required fields,
                                          event table, decision log)
                                        - GLOBAL-TEST-001 (principles,
                                          tooling, coverage thresholds,
                                          IT-001–IT-006)
                                        - GLOBAL-INFRA-001 (complete
                                          monorepo tree, Docker Compose,
                                          CI pipeline, release pipeline,
                                          Turborepo config)
                                        - OPERATOR-001 (first-time setup,
                                          start commands, incident recovery,
                                          nightly checklist)
                                        - GLOBAL-OPEN-001 (11 open items)
                                        - GLOBAL-CHANGELOG (this file)

                                        Meta commentary removed:
                                        - "What was completed & why"
                                          section removed (was not spec
                                          content; was assistant output)
Part 3 is complete. Here is a summary of everything covered:

✅ Stage 6 — Wiki — Full FR + AC: trigger model, sensitivity articles (with anti-vagueness rule), batch summary, interaction articles, topology articles, KG summary update, DB write, git commit
✅ Stage 7 — Knowledge Graph — Full FR + AC: raw observation insert, sensitivity distribution update, dead zone detection + clearing, hot zone detection + clearing, Mann–Whitney U interaction discovery, detect-impact implementation, GPR surrogate training
✅ Stage 8 — GEP — Full FR + AC: population=16, elites=top-2, selection breakdown 2+4+8+2=16 (consistent throughout), genome schema, fitness (history-only), crossover (uniform), single-gene mutation bounds, checkpoint recovery, 11 acceptance criteria covering every FR
✅ Stage 9 — Swarm — Full FR + AC: node identity, registration, heartbeat, 8-region partition table, SETNX claim protocol, region validation guard, shared registry push, global best M sync, OFFLINE_MODE + reconnect, conflict resolution, clean shutdown, dashboard feed
✅ GLOBAL-INFRA-001 — Complete monorepo tree (every file named), Docker Compose services table, CI pipeline (6 jobs), release pipeline, Turborepo turbo.json
✅ OPERATOR-001 — First-time setup, single-node start, swarm start, all CLI commands, 6 incident recovery procedures with exact commands, nightly checklist
✅ GLOBAL-OPEN-001 — 11 open items with resolution proposals
✅ GLOBAL-CHANGELOG — Full history from v0.1 to v1.0 with every consistency fix catalogued


