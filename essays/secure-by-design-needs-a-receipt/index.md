---
layout: essay
title: "Secure by design needs a receipt"
description: "A security-practitioner argument for guided design, local gates, and signed receipts before AI-assisted implementation hardens around weak choices."
date: 2026-05-20
last_updated: 2026-05-20
---

Secure by design needs a receipt.

I mean that literally.

Not a pledge. Not a scanner finding after the architecture settles. A small artifact in the repo that records the security decision before implementation hardens around it.

What was chosen. What was rejected. Which standard supports the choice. What local gate enforces it. Who accepted the risk.

In February 2025, CISA published a [Secure by Design Alert on eliminating buffer overflow vulnerabilities](https://www.cisa.gov/resources-tools/resources/secure-design-alert-eliminating-buffer-overflow-vulnerabilities) calling buffer overflows an "unforgivable defect." That was the right word. This class of bug is old. The safer patterns are known. The cost of choosing well at design time is lower than the cost of finding the same mistake after release.

Then the public record kept filling up.

In the roughly three and a half months after the alert, the National Vulnerability Database recorded 534 CWE assignment hits across three buffer-overflow-related weaknesses. The comparable pre-alert window had 527. De-duplicated, the count was 532 unique CVEs after the alert versus 524 before it.[^nvd]

| Weakness | Pre-alert | Post-alert |
|---|---:|---:|
| [CWE-787 Out-of-bounds Write](https://cwe.mitre.org/data/definitions/787.html) | 362 | 257 |
| [CWE-120 Buffer Copy without Checking Size](https://cwe.mitre.org/data/definitions/120.html) | 30 | 124 |
| [CWE-125 Out-of-bounds Read](https://cwe.mitre.org/data/definitions/125.html) | 135 | 153 |
| **CWE assignment hits** | **527** | **534** |

![Grouped bar chart showing NVD CWE assignment hits for CWE-787, CWE-120, and CWE-125 before and after CISA's February 2025 buffer-overflow alert.](/assets/img/secure-by-design-needs-a-receipt/nvd-cwe-pre-post-bars.svg)

*Comparable windows, not a causality claim. The point is narrower: warning alone is not a control.*

That does not prove the alert failed. CVE intake is noisy, delayed, and shaped by cleanup work. It does show why awareness is such a thin control.

[Joint CISA/NSA/FBI memory-safe roadmap guidance](https://www.cisa.gov/sites/default/files/2023-12/The-Case-for-Memory-Safe-Roadmaps-508c.pdf) cites this same class at roughly 70 percent of Microsoft's CVEs from 2006 to 2018, roughly 70 percent of Chromium vulnerabilities, and 67 percent of the 2021 zero-days analyzed by Google Project Zero.

The warning was correct. The missing object was still missing.

[^nvd]: Counts pulled from the [NVD CVE API 2.0](https://nvd.nist.gov/developers/vulnerabilities) on 2026-05-19 and rechecked on 2026-05-20. Query template: `cweId=<CWE-ID>&pubStartDate=<YYYY-MM-DD>T00:00:00.000&pubEndDate=<YYYY-MM-DD>T23:59:59.999`. Pre-alert window: 2024-10-28 to 2025-02-11. Post-alert window: 2025-02-12 to 2025-05-31. A CVE can carry more than one CWE. The table totals are additive CWE query hits, not unique CVEs.

## The category that almost exists

The field is not empty. Security already has the patterns: threat modeling, policy as code, SBOMs, build provenance, secure design reviews, paved paths, ADRs, signatures.

The problem is that they live in different rooms.

- **Threat modeling:** [Threagile](https://github.com/Threagile/threagile), [OWASP pytm](https://github.com/OWASP/pytm), [AWS Threat Composer](https://github.com/awslabs/threat-composer), [OWASP Threat Dragon](https://github.com/OWASP/threat-dragon)
- **Policy enforcement:** [OPA](https://github.com/open-policy-agent/opa), [conftest](https://github.com/open-policy-agent/conftest), [CUE](https://github.com/cue-lang/cue)
- **Paved paths and templates:** [Backstage](https://github.com/backstage/backstage), [Score](https://github.com/score-spec/score)
- **Software ingredients:** [Syft](https://github.com/anchore/syft), [CycloneDX](https://github.com/CycloneDX), [Dependency-Track](https://github.com/DependencyTrack/dependency-track)
- **Build attestation:** [in-toto](https://github.com/in-toto/in-toto), [Witness](https://github.com/in-toto/witness), [SLSA](https://slsa.dev), [Sigstore](https://github.com/sigstore)

Those are not failed tools. They are the pieces.

The gap is that these pieces have not been joined into a common OSS workflow for AI-assisted development: something that helps a developer move from intent to a right-sized security decision, then emits a repo-resident, standards-mapped, signed packet with generated gates and review evidence.

That is the missing category: older security muscle packaged at the exact point where AI turns intent into code.

## The decision under the bug

Most production vulnerabilities are downstream of an earlier design decision.

SQL injection usually starts earlier than string concatenation. Someone failed to make the safe data-access path the easy path.

Credential exposure usually starts earlier than a secret-shaped string in Git. Someone failed to decide where secrets may exist, who may read them, and what a tool is allowed to see.

By the time those failures show up in code, the path of least resistance has already been paved. Usually by accident. We should at least have the decency to pave it on purpose.

AI-assisted development compresses the distance between idea and code, which is the appeal and the risk in the same motion. A developer can ask an assistant to wire up auth, write a local tool, or scaffold a review bot. The result may be useful. It may also cross boundaries the team never named.

Agentic coding tools make that sharper. [NVIDIA's AI Red Team guidance on sandboxing agentic workflows](https://developer.nvidia.com/blog/practical-security-guidance-for-sandboxing-agentic-workflows-and-managing-execution-risk/) names the obvious risk: AI coding agents run tools from the command line with the user's permissions, which expands the attack surface on the developer machine. [Simon Willison's lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) gives the cleaner AI-specific version: private data access, exposure to untrusted content, and external communication in the same agent create an exfiltration path.

![xkcd 1200, Authorization, by Randall Munroe.](/assets/img/secure-by-design-needs-a-receipt/xkcd-1200-authorization.png)

*[xkcd 1200, "Authorization"](https://xkcd.com/1200/) by Randall Munroe, licensed [CC BY-NC 2.5](https://creativecommons.org/licenses/by-nc/2.5/). The joke is the boundary problem: the protected thing is not always the thing that can hurt you.*

That is a design decision before it is a scanner finding. Which leg is cut? Where is that decision recorded? Who has to look at it before a model gains new tool access?

If the answer is "we told the model not to," you do not have a security control. You have a wish with syntax highlighting.

## Guided design, not paperwork

The receipt is not asking a developer to already know the answer. It is what survives after the tool helps them find one.

That distinction matters. The goal is not an expert filling out better paperwork. The goal is a guided security design tool that helps a builder move from intent to a right-sized security decision before code exists.

The builder might be a lawyer trying to spin up a small internal app. It might be a young startup team without a security staff. It might be a security enablement team trying to make safe choices cheaper for everyone else.

![The Far Side, Midvale School for the Gifted, by Gary Larson.](/assets/img/secure-by-design-needs-a-receipt/the-far-side-midvale-school-for-the-gifted.png)

*The Far Side, "Midvale School for the Gifted", by Gary Larson. It works here because effort is not the same thing as a usable path.*

The tool should ask the questions security teams usually ask too late: who can use it, what data it touches, where secrets live, what the model can see, what it can change, which paved paths fit, and what gates prove the boundary held.

The output is the receipt.

![Flow diagram showing builder intent moving through guided security questions, category decisions, standards mapping, generated gates, review evidence, and a signed packet.](/assets/img/secure-by-design-needs-a-receipt/intent-to-signed-packet-chain.svg)

For example, the receipt for a synthetic AI code-review assistant might say:

```yaml
system: ai-code-review-assistant
accepted_path: "read repo content, draft PR feedback, require human approval to post"
rejected_paths:
  - "post comments externally without approval"
  - "read local secret stores"
cut_leg: external_communication
standards:
  - OWASP-LLM06
  - NIST-SSDF-PW.1.2
gates:
  - no_network_by_default
  - no_secret_paths_in_context
review: security-signoff-required
```

That is small enough to review, diff, and sign. It is also concrete enough for CI to refuse when the implementation drifts away from the decision.

For an AI-bearing application, the packet asks specific questions:

- `data-handling`: what data classes the system touches, where data is allowed to go.
- `ai-tool-boundaries`: which model/tool actions are allowed, which Lethal Trifecta leg is deliberately cut.
- `identity-auth`: who can use the system, what identity model applies.
- `secrets-egress`: what secrets exist, what redaction occurs before the model boundary.
- `logging-audit`: what evidence survives after the chat is gone.
- `implementation-hygiene`: what language-boundary patterns are forbidden.

Each decision carries a `provenance` field naming origin: deterministic-resolver, developer-authored, developer-LLM-assisted, or tool-LLM-generated.

[NIST SP 800-218](https://csrc.nist.gov/pubs/sp/800/218/final), the Secure Software Development Framework, gives the secure-development vocabulary that the rest of the federal AppSec stack cross-references. Two of its tasks land where this essay sits: PW.1.1 is threat modeling, and PW.1.2 is tracking the software's security requirements, risks, and design decisions. The category packs map onto PW.1.1; the signed packet is the operational form of PW.1.2.[^ssdf-tasks] OWASP gives the application and LLM failure modes. LLM06 Excessive Agency lives in the `ai-tool-boundaries` category. CISA gives the secure-by-design posture. The [in-toto attestation spec](https://github.com/in-toto/attestation) gives the envelope shape: a Statement binds metadata to a subject artifact, and a predicate says what kind of claim is being made about it.[^slsa]

[^ssdf-tasks]: NIST SP 800-218 (SSDF v1.1), Table 1. PW.1.1 calls for risk modeling (threat modeling, attack modeling, or attack surface mapping) to assess the software's security risk. PW.1.2's Example 2 reads: "Maintain records of design decisions, risk responses, and approved exceptions that can be used for auditing and maintenance purposes throughout the rest of the software life cycle."

[^slsa]: SLSA v1.1 is useful for build provenance. The design claim here is narrower. The packet says what the team decided. SLSA provenance says how the artifact was built. They sit side by side.

## Honest AI applied to the SDLC

This is where the [Honest AI](/essays/honest-ai/) argument meets the SDLC.

Citation chains become resolved standards IDs. Calibration becomes deterministic replay. Non-repudiation becomes accountable review. Trace review becomes a gate plan and evidence bundle. Structural defense becomes a policy the model cannot rewrite.

The same habits move earlier. They belong at the moment intent turns into implementation.

A stricter prompt is still a request. A gate is a control. The control has to live outside the prompt.

## Why local and deterministic matters

The foundation should be local and deterministic.

The packet should be verifiable from frozen inputs: the packet itself, the tool version, the bundled standards catalog, the policy bundle, and the signature material. Vendor APIs, hosted models, live standards lookups, and transparency services can add distribution and shared trust later. They should not be required to verify the packet.

The initial release has a narrower job: produce and verify a signed packet without depending on any external service.

The core must be deterministic:

```text
same frozen input
+ same tool version
+ same bundled standards catalog
+ same policy bundle
= same normalized packet
+ same validation result
+ same gate plan
+ same deterministic evidence
```

LLMs can still help author the packet. They are part of the modern developer workstation now, like editors and terminals.[^untrusted-actor] But their output has to freeze into an input before validation. The tool should not need to ask the model again to decide whether the same packet is valid.

[^untrusted-actor]: The intellectual lineage is older than software. Intelligence analysis tradecraft, where I started my career in the USAF and later at NSA, treats every source as potentially unreliable. The discipline is to assess confidence, fuse multiple sources, label assertions with provenance, and brief decisively while keeping the chain of evidence reconstructable. The same discipline transfers to AI-assisted engineering with one renaming.

That is the boundary:

```text
LLM use may affect authoring.
LLM use must not affect deterministic validation unless its output is frozen as input.
```

The same principle applies to signing. A signature binds the frozen packet digest and review evidence via a local ed25519 key. It should not mean "the model would probably say the same thing if we reran the prompt."

Rerunning a prompt is not provenance. It is a fresh dice roll wearing a lab coat.

## The scanner still belongs

Scanners are useful. They are also late.

A scanner can flag that `shell=True` showed up in a Python subprocess call. It cannot tell you why the team allowed shell boundaries in product code. It can detect a secret-shaped string. It cannot tell you whether the architecture ever should have let the AI assistant near the directory where secrets live. [Semgrep's own README](https://github.com/semgrep/semgrep) is honest about the scope: the tool "can only analyze code within the boundaries of a single function or file." Cross-file, cross-function reachability is the paid tier.

Without a design record, every scanner finding becomes a local argument. Is this a real risk? Was this path intentional? Does the team have an exception process?

With the record, the scanner becomes part of a chain:

```text
decision -> gate -> evidence -> review -> attestation
```

That is the work product security teams should want. Not a pile of findings. A chain of custody for engineering intent.

## A small worked example

The cleanest demo target is this project itself: a synthetic AI code-review assistant with enough risk to make the receipt useful and enough constraint to keep the example honest.

The demo assistant reads repo content, reads issue and design text, drafts PR comments, and must not post comments externally without human approval. Running `secure-path draft` produces a packet with six categories populated and unresolved decisions flagged for human review. A reviewer resolves them, signs off, and runs `secure-path attest --sign`.

The system refuses to sign in four cases that matter:

- An unresolved decision remains.
- A blocking finding is unresolved.
- The packet was modified after the audit review.
- A packet claims cross-model audit but no cross-model audit evidence exists.

The pattern is the [in-toto attestation spec's monotonic principle](https://github.com/in-toto/attestation): prefer "deny unless a no-vulnerabilities attestation exists" over "deny if a has-vulnerabilities attestation exists." Ignoring an attestation can never flip a deny into an allow. Refusal is the floor.

Each refusal is deliberate. [Anthropic's August 2025 threat-intel report](https://www.anthropic.com/news/detecting-countering-misuse-aug-2025) is useful here because the failure was workflow, not model output in isolation.[^anthropic-vibe]

| Case | What matters for the harness | Source hook |
|---|---|---|
| GTG-2002 extortion campaign | Operational instructions lived in a `CLAUDE.md` file, giving every Claude Code session the same persistent playbook. | At least 17 organizations affected; ransom demands ranged from $75,000 to $500,000. |
| GTG-5004 ransomware-as-a-service | Claude helped produce ransomware sold in tiers. | Pricing ranged from $400 to $1,200. |
| Chinese threat actor campaign | Claude supported activity across 12 of 14 MITRE ATT&CK tactics. | The report describes a 9-month operation against Vietnamese telecom and government infrastructure. |

The design lesson is narrower than the threat-intel report. If an agent can read and act on mutable local instructions, those instructions are part of the security boundary. Can the agent read its own operating instructions? Can it modify them? Can it carry them across sessions? What local gate proves the answer?

NVIDIA's sandboxing guidance gives the defensive mirror. It calls out writes to hooks, skills, and local agent configuration as a control boundary, because those files can change what an agent does before a human sees the next action.

That gives us another receipt example:

```yaml
system: local-agent-workflow
decision: "agent may read project config but may not modify agent-control files"
protected_paths:
  - ".claude/"
  - ".codex/"
  - ".cursor/"
  - ".github/workflows/"
  - "hooks/"
gates:
  - deny_agent_config_writes
  - require_human_review_for_control_plane_changes
evidence:
  - ci_path_policy_result
  - review_signature
```

If those questions are answered only in a chat transcript, they are not answered.

The whole demo runs with networking disabled. The CI test that enforces this is the proof.

[^anthropic-vibe]: Anthropic Threat Intelligence Report, August 2025. GTG-2002 specifics come from p. 4, including at least 17 organizations affected, ransom demands ranging from $75,000 to $500,000, and the `CLAUDE.md` persistent-context detail. GTG-5004 pricing comes from p. 15. Chinese threat actor campaign and MITRE ATT&CK coverage come from p. 17-18.

## Limitations

The initial release is intentionally small: six categories, two internal decision spines, four built-in checks, and one demo archetype. The point is a credible thread through one engineering practice: guided security design before implementation, followed by a signed packet and enforceable gates. The roadmap covers a full PR review gate, a design-to-PR review chain, multiple archetypes, and parallel category-pack agents for later versions.

I have not reviewed every commercial AppSec platform. SD Elements, IriusRisk, and similar tools work near this space. The gap I care about is narrower: a guided tool that can help the developer produce a repo-native signed decision artifact before implementation and enforce it in CI.

The research does not support panic. It supports humility.

In [Perry, Srivastava, Kumar, and Boneh's CCS 2023 study](https://arxiv.org/abs/2211.03622), participants with access to an AI code assistant wrote less secure code in four of five tasks and were more likely to believe their insecure code was secure. [AgentSafetyBench](https://arxiv.org/abs/2412.14470) is newer and preprint-stage, but the signal rhymes: 16 popular LLM agents, 349 environments, none above 60 percent on safety.

I have also seen the pattern in my own `framework-bench` work, but that deserves its own writeup. This essay should stand on the public evidence: published AI-assistant studies, agent-safety benchmarks, Anthropic threat intel, Embrace The Red incident writeups, and Trail of Bits' MCP work. The pattern is consistent. The frequency at which it shows up in commercial codebases is not something I can publish.

The private source cache for this essay contains 230 local reference files. It is a research artifact, not a public dependency.

## The real product is a habit

The habit is not paperwork. It is guided security design before implementation.

A developer should be able to describe what they are trying to build and have the tool ask the questions security teams usually ask too late: who can use it, what data it touches, where secrets live, what the model can see, what it can change, which paved paths fit, and what gates prove the boundary held.

The output is the receipt: the decision made, the paths rejected, the standard behind the choice, the gate that enforces it, the evidence that survived review.

Then let the scanners do what they do well.

The scanner is useful. It is not the architect.
