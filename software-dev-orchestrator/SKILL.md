---
name: 软件开发总调度
description: 软件开发总调度(Full-pipeline orchestrator). Runs the spec-driven workflow end to end — writes the spec (spec-driven-development), implements it with correct architecture (java-springboot, springboot-patterns, java-junit for tests), then reviews the result on both axes (code-review: Standards + Spec). Use when the user asks to "run the full pipeline", "spec then implement then review", wants a feature delivered with spec traceability, or says "软件开发总调度", "总调度", "spec-to-review". Also use when the user describes a requirement and wants the complete loop: clarify → spec → plan → implement → review, with human gates between phases.
---

# 软件开发总调度 (Software Dev Orchestrator)

Orchestrates the full delivery loop by dispatching to existing skills at each phase.
This skill owns **phase transitions and gates only** — all phase content comes from the dispatched skills.

## Pipeline overview

```
Phase 0-1  SPEC      → dispatch: spec-driven-development
Phase 2    DESIGN    → dispatch: writing-design-docs        (设计文档)
Phase 3    PLAN+TASKS→ dispatch: spec-driven-development
Phase 4    IMPLEMENT → dispatch: java-springboot + springboot-patterns + java-junit
                      + spring-boot-openapi-documentation   (接口文档)
Phase 5    TESTCASES → builtin template + java-junit        (测试用例文档)
Phase 6    REVIEW    → dispatch: code-review → builtin record template (评审记录)
```

Every phase transition requires explicit human approval. Never advance silently.

## Phase 1: SPECIFY (dispatch → spec-driven-development)

Load `spec-driven-development` and run its Phase 0 (capability map, if the request
bundles several independently testable capabilities) and Phase 1 (Specify):

1. Follow that skill's clarification loop: surface assumptions, ask questions,
   reframe vague requirements as testable success criteria.
2. Write the spec using its six-area template (Objective / Commands / Structure /
   Style / Testing / Boundaries) and save as `SPEC.md` (or `SPEC-<module>.md` per
   its capability-map convention).
3. Summarize, list Open Questions, then **STOP and ask the user to approve**.

**Gate: do not proceed until the user approves the spec in a later message.**

## Phase 2: DESIGN (dispatch → writing-design-docs)

After spec approval, load `writing-design-docs` and produce the technical
design document. Follow that skill's proposal skeleton (Why & What decision
box, completeness checks) covering:

- Architecture: components, layers, and their interactions (align with
  `springboot-patterns` layered conventions)
- Data model: entities, relationships, storage decisions
- Interface contracts: endpoints/DTOs at boundary level (detail comes in Phase 4)
- Risks, alternatives considered, and migration/compatibility notes

Save as `docs/DESIGN.md` (or `DESIGN-<module>.md`). **STOP for approval.**

**Gate: do not proceed until the user approves the design.**

## Phase 3: PLAN + TASKS (dispatch → spec-driven-development)

After design approval, continue with `spec-driven-development` Phase 2-3:

1. Generate the implementation plan (components, dependencies, build order, risks)
   → save to `tasks/plan.md`.
2. Break into tasks (each ≤ ~5 files, with Acceptance / Verify / Files fields)
   → save to `tasks/todo.md`.
3. **STOP and ask the user to approve the plan.**

**Gate: do not proceed until the user approves the plan.**

## Phase 4: IMPLEMENT (dispatch → java-springboot + springboot-patterns + java-junit + spring-boot-openapi-documentation)

Execute approved tasks one at a time. At each task:

1. **Architecture compliance** — load `springboot-patterns` before writing any
   class: follow its layered architecture (Controller → Service → Repository),
   REST design, and data-access conventions. Load `java-springboot` for
   Spring Boot framework best practices.
2. **API documentation** — load `spring-boot-openapi-documentation` when a task
   adds or changes REST endpoints: apply SpringDoc OpenAPI 3.0 annotations,
   Swagger UI config, request/response schemas, and security schemes. Keep
   the generated OpenAPI spec in sync with the code.
3. **Write tests** — load `java-junit` for JUnit 5 conventions when the task's
   Verify step involves tests; write tests per the spec's Testing Strategy.
4. Implement the task, run its Verify step, then move to the next task.
5. After every task: tick it in `tasks/todo.md` and report one-line status.

**Gate: after all tasks, ask the user to confirm before test-case documentation.**
If any task fails verification twice, stop and report instead of improvising.

## Phase 5: TEST CASES (builtin template, content via java-junit)

Produce a standalone test-case document from the implemented behavior.
Load `java-junit` for JUnit 5 test-design conventions, then fill this template
and save as `docs/TEST-CASES.md`:

```markdown
# Test Cases: [Feature Name]
Source: SPEC.md / DESIGN.md §[section]

## TC-001 [用例名]
- 模块: [Controller/Service/Repository]
- 前置条件: [...]
- 步骤: 1. ... 2. ...
- 输入: [...] (含边界值: 空/极大/非法)
- 预期结果: [...] (可断言)
- 优先级: P0/P1/P2  状态: PASS/FAIL/未执行
```

Coverage rules: every Success Criterion in SPEC.md maps to ≥1 test case;
every API endpoint gets ≥1 normal + ≥1 abnormal case; boundary values are
mandatory for numeric/string inputs. **STOP for approval.**

## Phase 6: REVIEW (dispatch → code-review + builtin record template)

1. Identify the fixed point: the commit/branch state before Phase 4 began
   (suggest the user commit first if everything is uncommitted; review needs a diff).
2. Load `code-review` and run its full two-axis process:
   - **Standards axis** — its smell baseline + any repo coding standards +
     the layered-architecture rules from `springboot-patterns`.
   - **Spec axis** — the approved `SPEC.md` / `docs/DESIGN.md` / `tasks/plan.md`
     from Phases 1-3 is the spec source (highest priority per that skill's
     lookup order). `docs/TEST-CASES.md` execution results are additional evidence.
3. Present both reports side by side, then a remediation list.
4. **Record the review** — persist the outcome as `docs/REVIEW-LOG.md`,
   appending one entry per review round:

```markdown
# Review Log
## Round N — [date]
- 审查基线: <fixed-point>...HEAD (commits: ...)
- Standards 轴: X 项发现(最严重: ...)
- Spec 轴: Y 项发现(最严重: ...)
- 结论: PASS / PASS-with-notes / FAIL(需修复后复审)
- 遗留问题: [编号+负责人+理由]
- 人工确认: [用户名/时间]
```

5. Fix accepted findings by looping back into Phase 4 mechanics (architecture
   skills for structural fixes, tests first for behavioral fixes). Re-run review
   (new round appended to the log) until both axes pass or the user accepts
   the findings.

## Phase map to dispatched skills

| Phase | Skill | What it contributes |
|---|---|---|
| 1 Specify | spec-driven-development | clarification, capability map, six-area spec, assumptions surfacing |
| 2 Design | writing-design-docs | design doc: Why/What decision box, architecture, completeness checks |
| 3 Plan/Tasks | spec-driven-development | plan structure, task sizing, verification steps |
| 4 Implement | springboot-patterns | layered architecture, REST conventions — prevents logic embedding |
| 4 Implement | java-springboot | Spring Boot framework best practices |
| 4 Implement | spring-boot-openapi-documentation | SpringDoc OpenAPI 3.0 annotations, Swagger UI, API schemas |
| 4 Implement | java-junit | JUnit 5 testing conventions incl. data-driven tests |
| 5 Test cases | builtin template + java-junit | test-case document (TC table) with coverage rules |
| 6 Review | code-review + builtin template | two-axis review + persisted review log |

## Rules

- **Never skip a gate.** If the user says "continue" without reviewing, summarize
  what they are approving and proceed — but a silent phase skip is forbidden.
- **Spec drift check.** If implementation reveals the spec was wrong, return to
  Phase 1 to amend the spec first (spec-driven-development's "Keeping the Spec
  Alive"), then resume. Never implement against a spec you know is outdated.
- **One task at a time.** No batching multiple tasks into one edit spree —
  that is exactly the "embedded messy business logic" this pipeline exists to prevent.
- **Dispatch, don't duplicate.** If a dispatched skill's instructions conflict
  with this file, the dispatched skill wins for its phase; this file only
  sequences them.
- **Non-Java stacks.** For non-Java projects, Phase 4 replaces the Java skills
  with whatever architecture/test skills are installed (or plain AGENTS.md
  conventions) — the spec→implement→review skeleton is unchanged. Phase 4's
  OpenAPI step applies only when the project exposes REST APIs.
- **Document artifacts.** The pipeline always produces: `SPEC.md` (需求),
  `docs/DESIGN.md` (设计), `tasks/plan.md`+`tasks/todo.md` (计划/任务),
  OpenAPI/Swagger (接口), `docs/TEST-CASES.md` (测试用例),
  `docs/REVIEW-LOG.md` (评审记录). These are deliverables, not byproducts —
  commit them with the code.

## Completion criteria

The pipeline is done when:
- [ ] Spec approved by human (Phase 1 gate)
- [ ] Design doc `docs/DESIGN.md` approved (Phase 2 gate)
- [ ] Plan + tasks approved by human (Phase 3 gate)
- [ ] All tasks implemented and verified (Phase 4)
- [ ] API documentation generated/synced (OpenAPI/Swagger, Phase 4)
- [ ] Test-case document `docs/TEST-CASES.md` written and approved (Phase 5)
- [ ] code-review run on the full diff; both axes reported (Phase 6)
- [ ] Review round appended to `docs/REVIEW-LOG.md` (Phase 6)
- [ ] Findings fixed or explicitly accepted by the user
