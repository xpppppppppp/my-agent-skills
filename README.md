# my-agent-skills

Agent skills collection for AI coding agents (OpenCode / Claude Code / Cursor / Codex ...).

## Skills

### software-dev-orchestrator (软件开发总调度)

A **full-pipeline orchestrator** that turns a vague requirement into reviewed, documented code by dispatching to specialized skills at each phase, with **human approval gates** between every phase:

```
Phase 1  SPEC        需求文档 SPEC.md            (spec-driven-development)   ⛔ gate
Phase 2  DESIGN      设计文档 DESIGN.md          (writing-design-docs)       ⛔ gate
Phase 3  PLAN/TASKS  tasks/plan.md + todo.md     (spec-driven-development)   ⛔ gate
Phase 4  IMPLEMENT   code + OpenAPI docs         (springboot stack skills)
Phase 5  TESTCASES   docs/TEST-CASES.md          (builtin template + junit)  ⛔ gate
Phase 6  REVIEW      two-axis review + REVIEW-LOG.md (code-review)           loop until pass
```

It exists to prevent the classic AI-coding failure mode: business logic embedded everywhere, no spec, no traceability. Every phase produces a committed artifact (spec / design / plan / API docs / test cases / review log), and the final review checks code against BOTH the spec and coding standards.

**Trigger phrases**: "软件开发总调度", "总调度", "run the full pipeline", "spec then implement then review" — or just describe a requirement and ask for the complete loop.

### Install

```sh
npx skills add xpppppppppp/my-agent-skills --skill software-dev-orchestrator
```

### Required companion skills

This skill is an **orchestrator** — it dispatches to these skills at runtime. Install them too:

```sh
npx skills add addyosmani/agent-skills@spec-driven-development      # Phase 1/3 spec + planning
npx skills add riekelt/technical-writer@writing-design-docs         # Phase 2 design docs
npx skills add github/awesome-copilot@java-springboot               # Phase 4 Spring Boot
npx skills add affaan-m/ecc@springboot-patterns                     # Phase 4 layered architecture
npx skills add giuseppe-trisciuoglio/developer-kit@spring-boot-openapi-documentation  # Phase 4 API docs
npx skills add github/awesome-copilot@java-junit                    # Phase 4/5 JUnit 5
npx skills add mattpocock/skills@code-review                        # Phase 6 two-axis review
```

> Non-Java project? Phase 4 skills are swappable — the spec → design → implement → test → review skeleton works for any stack (see SKILL.md "Non-Java stacks").

## License

MIT
