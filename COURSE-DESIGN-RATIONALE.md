# Course Design Rationale

**Course:** Practical Usage of Claude Code for DevOps Automation  
**Learning Path:** AIOps & DevOps  
**Audience for this document:** Course reviewers  
**Status:** Tiers 1 & 2 complete · Tier 3 in design  
**Maintainer:** SourceFuse Learning & Development  

---

## 1. Purpose of this document

This document records the pedagogical decisions behind the course — why each example was chosen, why each structural element exists, how the content hierarchy was designed, and what principles govern the callouts, prompting tips, and watch-out sections. It is written for reviewers who need to evaluate the course's design coherence, not for learners working through the content.

It is intentionally internal. The course itself does not reference or link to this document.

---

## 2. Course design philosophy

### 2.1 Why this course exists in the learning path

The AIOps & DevOps learning path covers two domains that are increasingly inseparable: **operational automation** and **AI-assisted tooling**. Most existing training on AI coding tools treats them as code generators — tools you prompt once and review the output. This course is built around a different and more accurate model: **Claude Code as a contextual participant** in DevOps workflows, not a one-shot generator.

This distinction is not cosmetic. An engineer who treats Claude Code as a generator will use it for first-pass boilerplate and then disengage. An engineer who understands it as a participant — one that maintains context across turns, correlates runtime signals with code it wrote, and produces output that improves with each feedback cycle — will extract ten times the value from the same tool.

The course's primary job is to install that mental model before the learner ever writes a prompt.

### 2.2 Target learner profile

The learner is an experienced DevOps engineer or SRE — someone who already knows what Terraform, GitHub Actions, Dockerfiles, and Kubernetes are. The course is deliberately not introductory. Learners who need a primer on CI/CD or IaC are directed elsewhere in the learning path first.

This constraint shapes everything: the examples do not explain what a Dockerfile is. The callouts do not define what OIDC means. The watch-out sections assume the learner understands the operational context of the failure modes being described. Content that would be appropriate for a junior engineer is actively excluded.

### 2.3 The five-step format and why it is fixed

Every example in the course follows the same five-step arc:

1. **Problem** — the realistic scenario, with specific context
2. **Naive approach** — what an engineer actually writes under time pressure, with named failure modes
3. **Claude Code interaction** — the full multi-turn dialogue, exact prompts and responses
4. **Refined output** — the final artifact, annotated with decision rationale
5. **Watch out** — what Claude Code cannot verify, what the engineer must still own

This format is fixed across all examples intentionally. The consistency serves two purposes. First, it gives learners a predictable cognitive scaffold — they know what each section will deliver before they read it, which reduces the cognitive overhead of navigating new content. Second, it encodes a transferable workflow: the five steps are not just a course structure, they are the actual working pattern the learner should apply to new tasks beyond the course.

The format was also designed with progressive complexity in mind. A Tier 1 example and a Tier 3 example use the same five steps — but the problem in Tier 3 is cross-layer, the naive approach is more subtly wrong, the interaction is longer, and the watch-out section is denser. The fixed format makes the escalation visible.

### 2.4 Why the diagrams precede the examples

The two interactive diagrams — the participation model and the pipeline integration model — are positioned before any worked examples. This is deliberate. The diagrams establish the mental model; the examples instantiate it.

A learner who sees the Dockerfile example without the diagrams has learned how to write a better Dockerfile with Claude Code. A learner who has seen the diagrams first understands that the Dockerfile example is a demonstration of the feedback loop in action at the Code stage of a pipeline. That framing is what makes the content transfer to tasks the course did not cover.

The diagrams are also the only interactive elements in the "foundations" section — 9 and 15 clickable nodes respectively, each with practitioner-level panel content. They are not decorative. They are the conceptual layer that the rest of the course hangs on.

---

## 3. Example selection rationale

### 3.1 Why these four examples for Tier 1

Tier 1 is scoped to single-file tasks — one file, one feedback loop, clear success criterion. The four examples were selected to maximise the distinctiveness of their failure modes while covering the most universally relevant DevOps artifacts.

| Example | File | Primary failure mode in naive approach |
|---------|------|----------------------------------------|
| 1 | Dockerfile | Silent security and performance debt — works, but dangerous |
| 2 | Terraform variables.tf | Weak interface — works, but imposes cognitive load on callers |
| 3 | GitHub Actions pipeline | False security and correctness — works on happy path, fails dangerously elsewhere |
| 4 | Shell script | False confidence — exits 0 on failure, provides no reliable signal |

Each failure mode is categorically different. The Dockerfile is a security problem. The variables.tf is a usability and maintainability problem. The pipeline is a correctness and security problem. The shell script is a reliability and observability problem. Covering four different failure mode categories in four examples gives learners a broader vocabulary for recognising when Claude Code's first-pass output needs additional scrutiny.

### 3.2 Why this sequence

The sequence is ordered by immediacy of risk and by cognitive complexity.

**Example 1 (Dockerfile)** opens because containerisation is universal — every DevOps engineer has written or reviewed a Dockerfile — and because the naive approach's failure modes are visually striking. Running as root, unpinned base image, no health check: these are recognisable problems that create immediate trust in the course's analytical approach.

**Example 2 (variables.tf)** follows because it demonstrates a different category of problem — one where the naive output is not dangerous, just weak. This teaches learners that Claude Code's value is not only about catching security issues; it also produces interfaces that scale to teams. It also introduces the convention-alignment pattern (Turn 3) which recurs throughout Tier 2.

**Example 3 (GitHub Actions)** is the most technically complex Tier 1 example — OIDC, matrix strategy, workflow_call, permission scoping. It is placed third because learners have now seen two examples of the five-step format and the feedback loop pattern, so they can engage with more complex content without the format itself being a distraction. The Turn 2 OIDC permissions bug is the most practitioner-valuable single exchange in the entire course.

**Example 4 (Shell script)** closes Tier 1 because it introduces the cross-platform portability problem (macOS vs Linux), which appears again in Tier 2 and Tier 3, and because it formally introduces the idempotency concept that becomes load-bearing in Tier 2's multi-step workflow examples.

### 3.3 What each example contributes that the others do not

This section documents the specific moments in each example that are pedagogically important and why — the observations that justify each example's inclusion beyond the high-level failure mode category.

---

#### Example 1 — Dockerfile

**Naive approach failure mode: silent accumulation, not immediate breakage.**
The naive Dockerfile is seven lines and exits cleanly. It builds, the container runs, the CI pipeline goes green. The six failure modes — running as root, `node:latest`, `COPY . .` before `npm install`, no `.dockerignore`, no `HEALTHCHECK`, dev dependencies included — none of them produce an immediate error. This is the pedagogical point of Step 2: the most dangerous Dockerfiles are the ones that work. Learners who only see examples where the naive approach crashes are not prepared for the real pattern, which is slow-accumulating debt.

**Turn 1: the `dumb-init` decision is the non-obvious move.**
Claude Code added `dumb-init` without being explicitly asked. The prompt specified "no root, cache-optimised, minimal image, healthcheck" — signal handling was not in the requirements. This demonstrates Claude Code reasoning about production implications that the engineer did not think to name. A Node process running as PID 1 without an init system does not handle `SIGTERM` gracefully, which means Kubernetes pod shutdowns cause ungraceful termination. The course uses this moment to show that Claude Code's value in the Plan stage includes surfacing implicit operational requirements, not just satisfying stated ones.

**Turn 2: the Trivy CVE is a real-world pattern handled with precision.**
The Turn 2 feedback includes both a Docker linter warning (JSON args form for `HEALTHCHECK CMD`) and a `micromatch` CVE. Claude Code's response to the CVE is the pedagogically significant moment: rather than generating an `overrides` block immediately, it first advises the engineer to run `npm ls micromatch --omit=dev` to determine whether the package is actually present in the production dependency graph. This is the distinction between a code generator (which would patch the CVE) and a contextual participant (which reasons about whether the fix is necessary at all). The `--omit=dev` check is the kind of advice an experienced colleague gives; a tool gives the fix.

**Turn 3: the `emptyDir` solution is not a Dockerfile change.**
When the `readOnlyRootFilesystem` constraint surfaces, the correct answer is not to modify the Dockerfile — it is to add a Kubernetes `emptyDir` volume mount. Claude Code produces the Kubernetes YAML, not a Dockerfile change, and explains why. This teaches the important lesson that a Dockerfile is one layer in a system, and that operational constraints at the cluster layer must be satisfied at the cluster layer. The boundary is the learning.

---

#### Example 2 — Terraform variables.tf

**Naive approach failure mode: cognitive load transferred to every caller.**
The naive `variables.tf` is not wrong — it is incomplete. Every variable is declared. None has a type, description, or validation. The failure mode is not that the module breaks; it is that every engineer who uses the module must read `main.tf` to understand the interface. That cost is invisible in the module itself and accumulates with every caller. The Step 2 framing — "a list of names, not a module interface" — is the pedagogical core. Reviewers should verify that this framing appears in the `warn` callout exactly, because it is the principle that makes the example generalisable beyond Terraform.

**Turn 1: the `sensitive = true` / state file distinction is deliberately drawn.**
Claude Code marks `password` as `sensitive = true` and then immediately notes that this does not encrypt the value in the state file. This two-part statement is the most important moment in Example 2's Turn 1 response. Marking a variable sensitive suppresses it from plan output — which is valuable — but engineers frequently believe it also secures the state backend, which it does not. The course states the distinction in Claude Code's own response so that the learner encounters the correct mental model at the same moment they see the pattern applied, not as a separate caveat in Step 5.

**Turn 2: cross-variable validation requires Terraform ≥ 1.3 — and Claude Code knows this.**
The `max_allocated_storage` variable's validation block references `var.allocated_storage` — a cross-variable reference that requires Terraform ≥ 1.3. Claude Code flags the version requirement unprompted and provides the alternative (`precondition` block on the resource) for teams pinned to an older version. This demonstrates Claude Code reasoning about the operational constraints of the toolchain version, not just the semantics of the code being written. The `null` short-circuit pattern in the validation condition — `var.max_allocated_storage == null || var.max_allocated_storage > var.allocated_storage` — is also worth noting: it handles the optional/nullable case correctly without requiring the engineer to understand the evaluation order.

**Turn 3: convention inference from a single reference file.**
The engineer shares one existing module file. Claude Code infers four distinct conventions: required variables before optional, descriptions end with a period, attribute ordering (type → description → sensitive → default → validation), and section comment headers. The engineer did not enumerate these conventions — they were inferred from the example. This is the most explicit demonstration in the course of the convention-reference pattern, and it is introduced here rather than in a Tier 2 example so that learners have a clear, contained reference point when the pattern appears implicitly in more complex contexts later.

---

#### Example 3 — GitHub Actions pipeline

**Naive approach failure mode: false correctness on the happy path.**
The naive pipeline goes green. Tests pass, image pushes. But it has six distinct failure modes: long-lived credentials, no `needs` dependency (so `build-push` runs in parallel with `test`), image pushed on every branch not just `main`, no caching, single Node version, not reusable. None of these produce a failure on a clean run to `main`. They surface in specific conditions: a security audit, a broken PR that still pushes an image, a Node version regression, a team trying to reuse the workflow. The Step 2 framing distinguishes between a pipeline that optimises for the first run being green and one that optimises for correctness across all future runs — a distinction experienced engineers recognise immediately.

**Turn 2: the OIDC permissions bug is the most practitioner-valuable single exchange in the course.**
The error message — "Resource not accessible by integration" — is one of the most misleading error messages in the GitHub Actions ecosystem. It points at credentials, but the cause is permissions. Specifically: when a `permissions` block is declared at the workflow level, GitHub Actions does not propagate it to jobs that declare their own `permissions` block — they must redeclare `id-token: write` explicitly. This behaviour was tightened in 2023 and is not prominently documented. Claude Code diagnosed it from one error message and one piece of context (the permissions block is at the workflow level, not the job level). The diagnosis is correct, the explanation of why it happens is accurate, and the fix is minimal and targeted. Reviewers evaluating the course's claim that Claude Code is a contextual participant rather than a code generator should examine Turn 2 of Example 3 as the strongest single piece of evidence.

**Turn 3: dual-mode secret resolution and `fromJSON` for matrix inputs.**
The `workflow_call` refactor introduces two non-obvious patterns. First, `secrets.aws-role-arn || secrets.AWS_ROLE_ARN` — the OR expression that resolves the correct secret name whether the workflow is called externally (kebab-case name from `workflow_call` secrets) or run standalone (UPPER_CASE name from repository secrets). Second, `fromJSON(inputs.node-versions || '["18.x","20.x"]')` — converting a JSON-encoded string input into an array the matrix strategy can iterate, with a fallback for standalone runs where `inputs.node-versions` is undefined. Both patterns are produced without the engineer requesting them — they emerge from Claude Code reasoning about the dual-invocation requirement stated in the Turn 3 prompt.

---

#### Example 4 — Shell script

**Naive approach failure mode: the most insidious of all four examples.**
The naive script exits 0 on an unhealthy deployment. Not a wrong answer, not a crash — false confidence. The three mechanisms that cause this are each distinct and worth naming for reviewers: first, the absence of `set -e` means `kubectl rollout status` can time out and the script continues; second, `curl` exit codes are not checked, so a failed health response is printed but not acted on; third, `echo "Deployment check complete"` fires regardless of outcome, providing a success-sounding final line to any log reader. The Step 2 `warn` callout synthesises this as "the worst kind of script: it provides false confidence." This framing is the most important sentence in Example 4 and should survive any future content revisions.

**Turn 2: the most technically precise diagnosis in any of the four examples.**
The Bash 3.2 vs 5.x arithmetic comparison behaviour inside `[[ ]]` is real, obscure, and frequently encountered on macOS by engineers who have Homebrew-installed Bash 5 but whose scripts resolve to `/bin/bash` (3.2) when invoked directly. Claude Code diagnosed this from one line of stderr (`[[: 120: syntax error: invalid arithmetic operator`) and a bash version number. The fix — splitting the mixed `[[ ]] || [[ -eq ]]` into a regex check followed by a `(( ))` arithmetic context comparison — correctly targets the specific incompatibility rather than defensively rewriting the entire argument validation section. The additional Bash version guard added in Turn 2 converts a cryptic arithmetic error into a clear "Bash >= 3.2 required" message, which is the correct engineering response: when you discover a platform incompatibility, make the error visible rather than just fixing it silently.

**Turn 3: two genuinely tricky shell patterns handled correctly.**
The first is `(( attempt++ )) || true` under `set -e`. Arithmetic expressions in `(( ))` exit with status 1 when the result is 0 (falsy in arithmetic terms). When `attempt` increments from 0 to 1, the result is 1 — truthy, no problem. But the pattern is fragile for any increment that produces a zero result, and `set -e` would silently abort the loop. The `|| true` idiom is the standard defensive pattern; Claude Code applies it and explains why in the response, making the idiom transferable rather than just present in the code. The second is the `kubectl auth can-i` permissions advisory — framed correctly as a warning to stderr, not a hard block. Claude Code explains unprompted why it is advisory: `kubectl auth can-i` returns true for cluster-admins even when the script makes no writes, so a hard exit would prevent the script from running in environments with broad permissions even when those permissions are appropriate. The advisory pattern — warn and continue — is the correct engineering decision, and seeing it reasoned through correctly is what makes it pedagogically valuable.

---

#### Example 5 — Full CI/CD pipeline with security scanning

**Naive approach failure mode: structural shape without integration logic.**
The naive pipeline has four named stages — build, scan, push, deploy — and therefore looks like a CI/CD system. But none of the jobs have `needs` dependencies, so they run in parallel. The scan job may run before the image exists. The deploy job may reference an image that has not been pushed yet. The Trivy scan uses `exit-code: 0`, meaning it never blocks anything regardless of what it finds. The pipeline succeeds at doing the wrong things in the wrong order. This is the Tier 2 version of the Tier 1 shell script failure mode — not a broken artifact, but a false-confidence system. The Step 2 `warn` callout frames it precisely: "It does not fail; it succeeds at doing the wrong things in the wrong order."

**Turn 1: three non-obvious decisions produced without prompting.**
First, `if: always()` on the Trivy report artifact upload — the scan report is uploaded even when the scan fails and blocks the pipeline. This is the correct behaviour: the report is the evidence for why the pipeline was blocked, and losing it on failure defeats the audit purpose. Claude Code applied this without being asked. Second, `provenance: false` on the ECR push — Docker Build Push Action v5 enables provenance attestations by default, which creates multi-platform manifests that cause ECR to return unreliable digest values. Disabling provenance produces a clean single-manifest image and a reliable digest. This is an ECR-specific incompatibility that is not obvious from the action documentation. Third, `needs: [scan, approve]` on the push job — both the scan passing AND a human approving are required independently. If the scan fails, the approve job never triggers, so the pipeline is blocked at two independent gates. Claude Code structured this correctly without the engineer specifying the failure mode they were guarding against.

**Turn 4: the OOMKilled diagnosis is the strongest in the example.**
The rollout failure log contains `Exit Code: 137` and `CrashLoopBackOff`. Exit code 137 is SIGKILL from the kernel OOM killer — the container exceeded its memory limit and was terminated. Claude Code identified this from the raw kubectl output without any additional context, named the root cause correctly (memory limit too low, not an application bug), distinguished it from the CrashLoopBackOff (which is a consequence, not the cause), and produced a rollback script that addresses the immediate service disruption while explicitly noting that the root cause requires a separate fix. This three-part response — identify, separate consequence from cause, address immediate vs. underlying — is what an experienced SRE does in a P1 incident. Seeing it modelled in a learning context is the pedagogical value.

**Turn 5: targeted convention fixes at the multi-file level.**
The engineer identifies two convention mismatches across the four files — a label value in the Kubernetes manifest and a log prefix pattern in the rollback script — and asks for only the changed lines, not full file regeneration. Claude Code produces exactly that: the two changed lines with context, and introduces a `log()` helper function in the rollback script that centralises the timestamp format so future additions automatically use the correct prefix. This is the targeted-fix pattern operating at the multi-file level, and the `log()` helper demonstrates that a targeted fix can include a design improvement (centralised formatting) without expanding scope beyond what was asked.

---

#### Example 6 — Multi-environment Terraform with remote state

**Naive approach failure mode: configuration difference as the only environment barrier.**
The naive structure has three `.tfvars` files pointing at a single local state file. The only thing preventing a prod outage from a mistyped `terraform apply` command is the engineer selecting the correct `.tfvars` file. There is no structural isolation between environments, no state locking, and no resource naming that distinguishes which environment a resource belongs to. The Step 2 `warn` callout frames this precisely: "it is not a system; it is a hope." This framing is important for reviewers because it articulates why multi-environment Terraform is an isolation problem, not a configuration problem — a distinction that motivates the entire structure of the example.

**Turn 1: the backend key injection pattern is the non-obvious architectural decision.**
The `backend.tf` is committed to the repo without a `key` value. The key (`payment-api/dev/terraform.tfstate`) is injected at `terraform init` time via `-backend-config`. This means the same committed file serves all three environments — the environment is never hardcoded in version control. Claude Code chose this pattern without being asked, and explained why: hardcoding the key in `backend.tf` would require either three separate backend files or a file that gets modified before each init, both of which are error-prone. The injection pattern is the correct engineering choice and it is introduced here because it is a prerequisite for the CI pipeline integration described in the example.

**Turn 4: the chicken-and-egg bootstrap problem is named and solved correctly.**
The `terraform init` failure — S3 bucket does not exist — is the canonical first-run problem with remote state backends. Terraform cannot initialise the backend until the backend resources exist, but you cannot use Terraform to create those resources because the backend is not yet initialised. Claude Code names this correctly ("the expected chicken-and-egg problem") and solves it with an AWS CLI bootstrap script rather than suggesting a workaround that would compromise the architecture (such as using local state for the first apply). The idempotency of the bootstrap script — using `aws s3api head-bucket` and `aws dynamodb describe-table` to skip creation if resources exist — is applied correctly and mirrors the shell script discipline established in Example 4.

**Turn 5: the `prevent_destroy` vs `deletion_protection` distinction is the most technically precise moment in the example.**
The engineer asks how to handle environment-differentiated lifecycle rules given that Terraform does not support dynamic lifecycle blocks. Claude Code's answer correctly identifies the constraint, explains why `prevent_destroy` is a Terraform client-side guard (removable by editing the config before running destroy), and proposes `deletion_protection = var.enable_deletion_protection` as an AWS API-level alternative that cannot be bypassed without a separate API call. This is a real architectural decision that experienced Terraform engineers debate — the course takes a clear position with a clear rationale. The default value of `true` for `enable_deletion_protection` in the new variable is deliberate: engineers must explicitly opt out of protection rather than opting in, which is the correct default posture for a module that will be used across environments.

---

#### Example 7 — Kubernetes deployment with HPA and PDB

**Naive approach failure mode: visible during the first node maintenance window.**
The naive workload is a Deployment and nothing else. Three replicas with no topology constraints, no PDB, no HPA, and no Service. The failure mode — all replicas on one node, node drained, four-minute outage — is exactly what happened. This is the Tier 2 version of a failure that is invisible in normal operations and catastrophic in the first operational event. The example is built around a real incident rather than a hypothetical, which grounds the solution choices in consequence rather than theory.

**Turn 1: <code>DoNotSchedule</code> is produced because the failure mode was named.**
The topology spread constraint uses `whenUnsatisfiable: DoNotSchedule` — the hard constraint that blocks scheduling rather than merely attempting spread. Claude Code chose this over `ScheduleAnyway` because the prompt described an outage caused by pod concentration: "all three replicas landed on the same node." A prompt that said only "spread pods across zones" would likely have produced `ScheduleAnyway`. The failure mode made the constraint choice unambiguous. This is the prompting principle articulated in the tip: naming the operational scenario you are guarding against produces more precise constraint choices than stating the desired behaviour abstractly.

**Turn 3: the <code>minAvailable</code> vs <code>maxUnavailable</code> explanation is the most technically precise moment in the example.**
The engineer asks Claude Code to explain the choice between the two PDB constraint types. The answer is correct and non-obvious: `maxUnavailable: 1` looks equivalent to `minAvailable: 2` at 3 replicas but behaves very differently at 10 replicas (HPA maximum). At 10 replicas, `maxUnavailable: 1` serialises all node drains — one pod at a time, very slowly. `minAvailable: 2` allows 8 simultaneous evictions — node maintenance proceeds quickly at scale. The operational rule derived from this (`minReplicas ≥ minAvailable + 1`) is a constraint that Kubernetes does not validate — it is the kind of operational knowledge that only surfaces when someone hits the problem in production. The course presents it proactively.

**Turn 4: the metrics-server error is the expected first-run failure for HPA.**
The `<unknown>` metrics and `the server could not find the requested resource (get pods.metrics.k8s.io)` error is the universal first encounter with HPA on a cluster where metrics-server has not been installed. Claude Code warned of this prerequisite in Turn 2; Turn 4 confirms it happened as predicted. The response includes both the standard installation and the EKS private endpoint TLS patch — the latter produced without prompting, based on Claude Code knowing that EKS private API endpoints commonly require it. This is the same pattern as Example 3 Turn 2: a specific failure mode diagnosed precisely from a short error excerpt.

**Turn 5: label alignment audit across five reference points.**
The Service, HPA, PDB, topology spread constraints, and Deployment pod template all reference the label `app: payment-api`. Claude Code produces a structured audit of all five reference points and names the failure mode if they drift: traffic not routed, autoscaling not applied, PDB protecting nothing, spread not enforced. This is the multi-file integration audit pattern at the Kubernetes manifest level — analogous to the Turn 5 convention check in Example 5, but operating on semantic correctness rather than naming conventions.

---

#### Example 8 — Argo CD application promotion flow

**Naive approach failure mode: automation without guardrails is worse than manual deployment.**
The naive Argo CD setup has `prune: true` and `selfHeal: true` on the prod Application, with `targetRevision: main`. This gives Git unlimited write authority over the production cluster with no human checkpoint. A wrong commit, a refactored directory, or an accidental file deletion becomes a prod incident — and the `selfHeal` flag means even a manual emergency rollback via `kubectl` is immediately reverted. The Step 2 `warn` callout articulates this precisely: "True GitOps for prod requires automation with guardrails, not automation without limits." This is the most consequential failure mode in any Tier 2 example — the naive approach is actively more dangerous than no automation at all.

**Turn 1: the conditional `automated` block is the most architecturally significant decision.**
The ApplicationSet template uses a Go template conditional `{{#if (eq autoSync "true")}}` to omit the `automated` block entirely for prod — not just set it to empty. This is correct: an `automated: {}` block with no flags still enables auto-sync with defaults in some Argo CD versions. The only safe way to disable auto-sync in an ApplicationSet template is to omit the block. Claude Code chose this without being prompted, and explained why in the response. This is a non-obvious Argo CD behaviour that is not clearly documented and has caused production incidents where teams believed auto-sync was disabled but the empty block was still triggering syncs.

**Turn 2: the `PLACEHOLDER` tag in the prod overlay is an intentional fail-safe.**
The prod overlay's `newTag: PLACEHOLDER` is not a documentation placeholder — it is a deployment-time guard. The string `PLACEHOLDER` is not a valid image tag; any attempt to apply the prod overlay without running the promotion script first will fail at the image pull step with a recognisable error. This pattern converts a process requirement (always run the promotion script) into a technical enforcement mechanism. The course explains this explicitly in the Turn 2 response, making the intent visible to anyone who reads the file.

**Turn 4: two-issue diagnosis from a single Argo CD status JSON.**
The status output shows both a `ComparisonError` (HPA `autoscaling/v2` not supported on the cluster version) and a `Degraded` health message (Deployment minimum pods not available). Claude Code correctly identifies that these are two connected issues — not two independent problems. The HPA API version incompatibility causes the HPA not to be applied, which causes the Deployment to be unable to reach its desired replica count, which causes the Degraded health. Naming the causal chain — not just listing two errors — is what distinguishes the diagnosis from a list of symptoms. The recommendation to verify the prod cluster version before the prod promotion (made at the end of Turn 4) is the kind of forward-looking advice an experienced SRE gives after a staging incident.

**Turn 5: the `REGISTRY` placeholder audit flags the one deployment prerequisite.**
The overlay audit is clean across all three structural checks. The only finding is the `REGISTRY` placeholder — not a structural problem but a deployment prerequisite. Claude Code flags it with the specific failure mode: Kustomize matches image names exactly, and a partial match silently skips the override. The word "silently" is the key: the promotion would appear to succeed, Argo CD would sync, and the cluster would continue running the old image. This is the same category of failure as the Example 5 container name alignment issue — a silent mismatch that produces no error but does the wrong thing.

---

#### Example 9 — Python automation script with retry logic

**Naive approach failure mode: a fixed sleep masquerading as retry logic.**
The naive script's `time.sleep(30)` is not retry logic — it is a fixed wait that is simultaneously too short (service not ready after 30 seconds) and too long (service ready after 5 seconds). The failure mode it produces is the same as Example 4's shell script: the script can exit 0 on an unhealthy state, and exit 1 on a healthy state that just needed more time. The Step 2 `warn` callout connects it explicitly to Example 4: "It produces false confidence in the same way as the naive shell script from Example 4." This cross-reference is deliberate — it reinforces the pattern that single-shot automation with no retry produces false confidence, regardless of the language.

**Turn 1: the module separation is the most architecturally significant decision.**
Claude Code produced `retry.py` as a standalone module rather than embedding retry logic in `smoke_test.py`. This was not explicitly requested — the prompt asked for "a reusable retry module" but did not specify whether it should be a separate file. Claude Code chose module separation because it is what makes the retry logic independently testable. A retry function embedded in the smoke test script can only be tested by running the full smoke test — which requires a running service. A separate module can be tested with unit tests against mocked responses, as demonstrated in Turn 3. The module separation is what makes the test-alongside-implementation pattern possible.

**Turn 2: the `main(argv=None)` pattern is the most important software engineering decision in Tier 2.**
The smoke test's `main` function accepts an optional argument list rather than reading from `sys.argv` directly. This single design decision makes the entire CLI testable without mocking `sys.argv` — the test can call `main(["--url", url, "--max-attempts", "5"])` directly. Claude Code introduced this pattern in Turn 2 and explained why it matters. It is one of the most commonly missed Python testing patterns, and its absence forces tests to use `unittest.mock.patch("sys.argv", [...])` — a fragile approach that couples tests to the implementation's internal use of `sys.argv`. The course presents the correct pattern first, without showing the incorrect alternative, because the example is not about testing anti-patterns.

**Turn 4: the root cause is a test design issue, not an implementation bug.**
The two test failures both trace to the `_SequenceHandler.call_count` class variable being shared across both endpoint checks within a single test invocation. Claude Code correctly identified this as a test design issue — the implementation being tested was working correctly. The diagnosis distinguishes between "the test is wrong" and "the code is wrong," which is a non-trivial distinction when the test output is a failing assertion. The fix is pragmatic (extend the response sequence) rather than architectural (refactor the mock server), which is correct for this scope — the architectural refactor is noted as a future consideration in the Watch Out section.

**Turn 5: contract tightening converts a silent ambiguity into a loud failure.**
The `is_retryable` function originally returned `False` for a 200 response — which is technically correct (a 200 is not retryable) but semantically wrong (the function should never be called with a 200). The Turn 5 change raises a `ValueError` instead, converting a silent API misuse into an immediate programming error. The `rng` optional dependency injection is a parallel fix — it converts a non-deterministic test into a deterministic one without affecting production behaviour. Both changes follow the same principle: make incorrect usage fail loudly rather than produce a wrong result silently.

---

## 4. Callout taxonomy

The course uses four callout types. Each has a specific pedagogical purpose. They are not used interchangeably.

### `info` (blue)
Used to present factual context the learner needs before engaging with the content — requirements, constraints, background. Never used for evaluative statements. Appears in Step 1 (Problem) to frame the scenario.

### `warn` (coral/red)
Used to deliver the core problem statement at the end of Step 2 (Naive approach). Always appears once per example, at the bottom of the naive approach section, as a synthesis of why the naive approach is specifically dangerous or insufficient. The tone is direct but not condescending — it names the exact failure mode and why it persists (usually: it doesn't fail immediately, so it goes undetected).

### `good` (green)
Used in Step 5 (Watch out) to acknowledge what Claude Code handled correctly before listing limitations. This is a deliberate structural choice: the watch-out section exists to be honest about Claude Code's limitations, but opening with criticism would undermine the learner's confidence in what they just built. The `good` callout resets the tone before the caveats.

### `tip` (amber)
Used at the end of Step 5 as the final element of every example. Always contains a specific, actionable prompting principle derived from the interaction that just occurred — not generic advice about prompting AI tools. The tip is deliberately tied to the specific example so learners can generalise it themselves rather than being told to generalise it.

---

## 5. Watch-out section design

### 5.1 Why limitations are explicit

The watch-out sections are the most unusual element of the course compared to standard AI tooling training, which typically omits or downplays limitations. The decision to include explicit, named limitations for every example was deliberate and is worth justifying.

The target learner is an experienced engineer. They will discover the limitations regardless — either in the course, or the first time they deploy something Claude Code helped write and it fails in an unexpected way. If they discover it in the course, they learn the limitation in a controlled context with an explanation. If they discover it in production, they lose trust in the tool entirely.

The watch-out sections are also what make the course credible to experienced practitioners. A course that only shows Claude Code succeeding reads as marketing. A course that names exactly what Claude Code cannot do — and why — reads as an honest engineering assessment.

### 5.2 Structure of each watch-out section

Every watch-out section follows the same internal structure:

1. A `good` callout acknowledging what Claude Code handled correctly
2. Three to five `Verify` annotation items — specific things the engineer must check that Claude Code cannot verify
3. One or two `Note` annotation items — contextual observations that are not failures but are important for completeness
4. A `tip` callout with the example-specific prompting principle

The `Verify` items are always specific to the learner's environment — things like IAM trust policies, registry configurations, kubeconfig permissions, provider version pins. These are not generic limitations of Claude Code; they are environment-specific facts that Claude Code has no visibility into. The distinction between "Claude Code got this wrong" and "Claude Code cannot know this" is important and is preserved in the language used throughout.

### 5.3 What the watch-out sections deliberately exclude

The watch-out sections do not include general AI disclaimers ("always review AI-generated code"), general security advice ("use least privilege"), or process reminders ("get a code review"). These are assumed knowledge for the target learner and would dilute the signal of the specific, actionable limitations.

---

## 6. Prompting tip design

Each example ends with a prompting tip derived from that specific example's interaction. The tips are not a general prompting guide — they are specific principles that the learner has just seen demonstrated.

| Example | Prompting tip principle |
|---------|------------------------|
| 1 — Dockerfile | Specify constraints explicitly (no root, cache-optimised, healthcheck) rather than intent ("production-grade") |
| 2 — variables.tf | Share existing module files as convention references — Claude Code infers conventions from examples better than from written style guides |
| 3 — GitHub Actions | Specify action versions explicitly in the prompt; specify OIDC role ARN source and region — these cannot be guessed |
| 4 — Shell script | Paste exact stderr output verbatim, not a description of the error — the exact tokens in an error message contain the diagnostic signal |

These four tips together encode the most important meta-lesson of the course: **the quality of Claude Code's output is directly proportional to the specificity and grounding of the input.** This lesson is not stated explicitly anywhere in the course — it is designed to be inductively discovered by the learner across the four examples.

---

## 7. Cross-example patterns

Three patterns appear in multiple examples and are worth naming explicitly for reviewers, because they represent the transferable skills the course is designed to build.

### 7.1 The paste-verbatim pattern

In every Turn 2 interaction, the engineer pastes exact output — build warnings, error logs, stderr — without paraphrasing. This is consistent across all four examples. The pattern teaches that paraphrased errors lose diagnostic signal and produce generic diagnoses, while exact output produces targeted fixes. This is demonstrated rather than stated because telling an experienced engineer to paste errors verbatim would feel condescending; showing the contrast between the two approaches makes the lesson self-evident.

### 7.2 The convention-reference pattern

Turn 3 of the Terraform example introduces the pattern of sharing existing repo files as convention references. This pattern recurs in Tier 2 — sharing existing pipeline YAML when generating a new one, sharing existing module structure when generating a new module. It is introduced explicitly in Tier 1 so learners recognise it when it appears implicitly in more complex Tier 2 contexts.

### 7.3 The targeted-fix pattern

Across all four examples, the Turn 2 and Turn 3 interactions do not ask Claude Code to regenerate the entire artifact. They ask for targeted fixes to specific identified problems. This is a deliberate counter-pattern to the common practice of regenerating from scratch when output is imperfect. The course models the correct behaviour — targeted correction with grounding context — consistently across all four Tier 1 examples so it becomes the learner's default working mode.

---

## 8. Diagram-to-example relationship

The two diagrams establish the conceptual framework. The four examples are the instantiation of that framework. The relationship is specific, not general.

**Diagram 1 (Participation model)** establishes that Claude Code has four input streams — repo context, runtime signals, infra state, policy context — and produces five output types. The examples instantiate this:

- The Dockerfile example uses repo context (existing conventions) and runtime signals (build output, Trivy scan)
- The variables.tf example uses repo context (existing module files as convention references) and policy context (security team requirements)
- The GitHub Actions example uses runtime signals (the OIDC error log from Turn 2)
- The Shell script example uses runtime signals (exact stderr from the macOS failure)

**Diagram 2 (Pipeline integration model)** establishes that Claude Code participates at every pipeline stage. The examples map to stages:

- Dockerfile → Code stage
- variables.tf → Plan stage
- GitHub Actions → Build & test stage
- Shell script → Deploy stage (pre-cutover verification)

This mapping is not made explicit in the course content — it is structural. Learners who internalise the diagrams will see the connection without being told. Reviewers evaluating the course's coherence should verify that each example genuinely demonstrates the stage-appropriate Claude Code behaviour described in Diagram 2's panel content for that stage.

---

## 9. Tier 2 and Tier 3 design intent

### 9.1 Tier 2 — Multi-step workflows

Tier 2 examples are distinguished from Tier 1 by three characteristics: multiple files are involved, the feedback loop spans multiple tools or pipeline stages, and the convention-reference pattern becomes implicit rather than explicit.

The five planned Tier 2 examples cover:

- **Full CI/CD pipeline with security scanning** — introduces pipeline architecture reasoning (parallelisation, approval gates) as distinct from syntax generation
- **Multi-environment Terraform with remote state** — introduces workspace patterns, backend configuration, and the interaction between module structure and environment separation
- **Kubernetes deployment with HPA and PDB** — introduces operational configuration beyond basic deployment manifests; the naive approach creates deployments that cannot be drained safely
- **Argo CD application promotion** — introduces GitOps patterns and the specific failure modes of automated sync in production (prune, selfHeal risks)
- **Python automation script with retry logic** — introduces the test-alongside-implementation pattern and exponential backoff as a design concern

### 9.2 Tier 3 — Context-heavy scenarios

Tier 3 examples are where the course bridges into AIOps. They are distinguished by cross-layer evidence (logs + config + cluster state simultaneously), novel failure patterns not covered by runbooks, and an explicit handoff to AIOps workflows at the Operate stage.

The four planned Tier 3 examples cover:

- **Broken deployment: cross-layer RCA** — the canonical multi-layer failure where application logs, Kubernetes events, and Terraform history must be correlated simultaneously
- **CI pipeline performance triage** — a pipeline that degraded over six months; requires analysing the full job log, YAML, dependency manifest, and cache config together
- **Incident RCA with AIOps handoff** — the natural endpoint of the DevOps module, where Claude Code's triage output becomes the input to AIOps remediation workflows
- **Alert enrichment and runbook generation** — the bridge between raw operational signals and structured, actionable documentation

### 9.3 The 70/30 DevOps/AIOps split

The course targets 9–10 DevOps automation examples and 3–4 AIOps-adjacent examples. This split was chosen because the AIOps module in the broader learning path covers incident analysis, RCA, and remediation patterns in depth. The AIOps-adjacent examples in this course are designed as a bridge — introducing the concepts and demonstrating the handoff — not as a substitute for the dedicated AIOps module.

---

## 10. Design decisions that may draw reviewer questions

### "Why not include a video or interactive simulation?"

The course is built on GitHub Pages as static HTML. The interactive elements — the clickable diagram nodes with panel content, the scroll-tracking step navigation, the copy buttons on code blocks — provide the interactivity appropriate to the medium without requiring a video production workflow or an LMS. The design prioritises content depth over production richness, which is appropriate for the experienced practitioner audience.

### "Why are the Claude Code interactions fictional rather than actual Claude Code runs?"

The interactions are carefully designed composites — they reflect real patterns of how Claude Code behaves, real failure modes it encounters, and real diagnostic capabilities it demonstrates, but they are not screen recordings of actual sessions. This was a deliberate choice for two reasons. First, actual Claude Code sessions produce variable output that may not illustrate the pedagogical point cleanly. Second, the course needs to remain accurate as the model evolves — composites based on demonstrated behaviour patterns age better than specific session transcripts.

### "Why is the watch-out section in Step 5 rather than integrated into the code annotations?"

Integrating limitations into code annotations (inline comments on the refined output) was considered and rejected. Annotations inside code blocks are read during code review, not during learning. Placing limitations in a dedicated Step 5 section ensures they are read as a complete set — learners see the full picture of what they need to verify before they move on, rather than encountering caveats piecemeal while reading code.

### "Why are prompting tips at the end of Step 5 rather than at the beginning?"

The prompting tip at the end of Step 5 works inductively: the learner has seen the interaction, seen the refined output, understood the limitations, and is now in the best possible position to absorb the meta-lesson about how to prompt. Placing the tip at the beginning (before the interaction) would be deductive — telling the learner what to notice before they have seen it. The inductive placement is more durable: principles learned from experience transfer better than principles memorised before experience.

---

## 11. Planned updates and known gaps

| Area | Current status | Planned |
|------|---------------|---------|
| Tier 2 examples (5) | Locked placeholder cards on course map | In design |
| Tier 3 examples (4) | Locked placeholder cards on course map | In design |
| Course map progress tracking | Static counter (2 of 11) | Dynamic via localStorage when examples added |
| Reviewer dashboard integration | Standalone `.md` | Link from sf-knowledge-hub reviewer dashboard |
| Light/dark mode for diagrams | Light mode only (matches course) | No change planned |
| Accessibility audit | Not yet conducted | Required before Tier 2 release |
| Mobile layout | Functional but not optimised | Optimise before full course release |

---

## 12. File inventory

All course files are hosted at `sflearningdevelopment.github.io` and stored in the course repository.

| File | Purpose |
|------|---------|
| `course-map.html` | Learner entry point — course navigation and tier overview |
| `claude-code-devops-diagram-1.html` | Interactive diagram — participation model (9 nodes) |
| `claude-code-devops-diagram-2.html` | Interactive diagram — pipeline integration model (15 nodes) |
| `example-1-dockerfile.html` | Tier 1, Example 1 — Dockerfile |
| `example-2-terraform-variables.html` | Tier 1, Example 2 — Terraform variables.tf |
| `example-3-github-actions-job.html` | Tier 1, Example 3 — GitHub Actions pipeline |
| `example-4-shell-script.html` | Tier 1, Example 4 — Shell script |
| `example-5-cicd-pipeline.html` | Tier 2, Example 5 — CI/CD pipeline with security scanning |
| `example-6-terraform-multienv.html` | Tier 2, Example 6 — Multi-environment Terraform |
| `example-7-kubernetes-hpa-pdb.html` | Tier 2, Example 7 — Kubernetes HPA and PDB |
| `example-8-argocd-promotion.html` | Tier 2, Example 8 — Argo CD promotion flow |
| `example-9-python-automation.html` | Tier 2, Example 9 — Python automation with retry logic |
| `COURSE-DESIGN-RATIONALE.md` | This document — internal reviewer access only |

---

*Document version: Tiers 1 & 2 complete. Updated as subsequent tiers are added.*  
*Not linked from the course map. Reviewer access via repository.*
