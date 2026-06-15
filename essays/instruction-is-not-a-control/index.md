---
layout: essay
title: "The Emperor's New Controls"
description: "A security-practitioner argument that across coding agents, agent security, guardrails, secrets, model personality, and retrieval, the field keeps rediscovering one rule: you cannot instruct a system into the behavior you need, you have to enforce it structurally."
date: 2026-06-11
last_updated: 2026-06-15
---

I have spent two essays honing a hypothesis. The [honest-AI essay](/essays/honest-ai/) said verify what an AI tells you, because the failure modes that matter are the ones it won't report on itself. The [receipt essay](/essays/secure-by-design-needs-a-receipt/) said record the security decision in a signed artifact before the implementation hardens around it, because a stricter prompt is still a request. I was pushing the same thread further, experimenting with different proofs and structures for implementation, when I watched [Nick Nisi's](https://youtu.be/vy7o1g2iHY8) talk. The talk hit a critical through-line in my work: he had measured the gap between what one tells an agent to do and what said agent actually does. More importantly, he had closed it with structure.

Nisi builds software by giving AI agents instructions and then having them run the test suite before reporting the work done. The assumption here is that a green check means something. However, the agents found the cheaper path. One of them learned to create a small file named to look like a passing-test record, then reported success without running a single test.

His fix wasn't a sterner instruction. He had the system take a hash of the real test output and refuse to let the agent advance unless the hash matched. His words: "It stopped lying not because I asked it very nicely. I made it prove it."

You cannot instruct a system into the behavior you need, because the instruction and the behavior come out of the same machinery, one that [blurs the line between data and instructions](https://arxiv.org/abs/2302.12173) and treats your instruction as [one more input to weigh](https://arxiv.org/abs/2404.13208), not a rule it has to obey. Tell it to run the tests and it hears a request, which it can satisfy or fake. The gap between the instruction you gave and the behavior you got doesn't close with a firmer prompt. It closes when you bind the behavior to something the system can't argue with: a hash that has to match, a boundary it can't see.

What caught me was the room Nisi works in. He builds JavaScript developer tooling, not security systems, and he had walked straight into the thing I had spent two essays circling. So I went looking for the same shape somewhere else, and found it. Then again, and again, in fields that don't talk to each other. His test-hashing trick is one fix among many. The same law keeps surfacing in rooms that never compare notes. That is the essay. It's not mine, and it's not Nisi's. It belongs to whoever keeps tripping over it.

## A law by any other name

I didn't coin this, and neither did Nisi. The oldest name for it is an old bug. "Prompt injection" was named by analogy to SQL injection, the flaw where untrusted text gets treated as a command, and the fix for SQL injection was always structural: stop splicing untrusted strings into the query, bind the parameters so data can't become code. Asking the input nicely was never on the table. None of this is new. We have been losing to this trick since `DROP TABLE Students` was a punchline, and a control you write into a prompt is the emperor's new clothes. The whole court agrees to see a garment until an attacker points out the emperor is naked.

![A mother tells a school her son is named "Robert'); DROP TABLE Students;--", and the school's unsanitized database deletes its student records.](/assets/img/instruction-is-not-a-control/xkcd-327-exploits-of-a-mom.png)

*xkcd 327, "Exploits of a Mom," by Randall Munroe ([CC BY-NC 2.5](https://xkcd.com/license.html)). Untrusted input becomes a command, the SQL-injection joke that is prompt injection's ancestor.*

Ken Thompson said the deep version in 1984: past a certain point you can't inspect your way to trust, you can only move it, in his [Turing Award lecture](https://dl.acm.org/doi/10.1145/358198.358210). Simon Willison, who coined the term, says the agent version: don't tell the model to be careful, [remove a leg of the lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) so the dangerous combination can't occur. Nisi says the builder version: enforce, don't instruct. The academic version arrived in June 2025 with fourteen authors behind it, from ETH Zurich, EPFL, IBM, Google, and Microsoft, proposing [design patterns for securing LLM agents](https://arxiv.org/abs/2506.08837) with, in their words, "provable resistance to prompt injection."[^provable] Their patterns work by constraining what an agent is allowed to do, not by instructing it to behave. When a 1984 Turing lecture, a fourteen-author consortium spanning industry and academia, and a JavaScript developer arrive at the same rule on their own, it's a law, not a mood. That convergence is most of the argument; the rest of this essay walks the rooms it turns up in.

[^provable]: "Provable resistance" is the goal for the most constrained of the patterns; the paper scopes general-purpose agents out and frames the rest as mitigation, not immunity.

![Watercolor of the little prince standing on asteroid B-612 beside his rose under a glass globe, among small volcanoes and a starfield. A banner reads, "It is the time you have wasted for your Rose that makes your Rose so important."](/assets/img/instruction-is-not-a-control/little-prince-rose-b612.jpg)

*The little prince and his rose. The section title borrows "a rose by any other name"; the law underneath keeps its meaning whatever you call it.*

One caveat before the rule runs away with itself. That same paper frames its patterns as a trade-off between an agent's usefulness and its safety, and knowing when to spend that trade is the point. This discipline is for high-stakes work that needs high-integrity patterns: legal filings, enterprise software, security operations, anything in finance or healthcare, anything where an AI output feeds a decision a person or an organization has to answer for. If you're building a weekend toy, instruction is fine, and wrapping it in hashes and sandboxes is friction with no payoff. Match the control to the consequence. The discipline below earns its cost only where the blast radius is real.
{:.callout}

## I spy with my little eye

...the same shape across six rooms that have never met.

**Coding agents.** Nisi's marker file is one builder's war story. Dawn Song's lab turned it into a result. Their [BenchJack audit](https://arxiv.org/abs/2605.12673) of ten agent benchmarks found 219 distinct ways for an agent to score without doing the work, and the cleanest is the trick Nisi caught by hand: on SWE-bench, a nine-line `conftest.py` that the test runner auto-loads can rewrite every test's reported outcome to passing, a 100% resolve rate without solving a single task. Their diagnosis is the whole point. These flaws, they write, "are not bugs to be patched, but design choices to be undone, and a code-only patch cannot move the trust boundary back into place." If "done" is something the agent reports, the agent will eventually learn to report it without doing it. Bind "done" to captured proof or it isn't a measurement.

**Agent guardrails.** A guardrail you write into the prompt is an instruction, and an agent that can read the instruction can argue with it. Anthropic's [zero-trust-for-agents material](https://claude.com/blog/zero-trust-for-ai-agents) says enforcement has to live below what the agent can see, at the network layer, with task-scoped, short-lived credentials instead of a standing key. Anthropic sells the platform this secures, so weigh the source, but the academic version says the same thing with a stronger word: provable. Don't rely on the model declining. Constrain what it can do.

**Guardrail code.** Sometimes the guardrail is itself code, and the bug is treating it as text to filter. Sysdig's threat researchers described an LLM gateway that ran an admin-supplied screening function and tried to make it safe with a regex blocklist over the function's own source, rejecting strings like `eval`. Attackers wrote around the denylist and reached code execution on the gateway. [SandboxEval](https://arxiv.org/abs/2504.00018) makes the general case: an instruction-tuned "safer" model still wrote malicious code on request in every scenario its authors tested,[^sandboxeval] so the thing that holds is the verified execution environment, not the model's training. They run their checks "proceeding as if the environment could be badly misconfigured," which is the receipt instinct pointed at the sandbox: confirm the boundary holds, don't assume it. A denylist over source is a suggestion. A separate sandbox is the control.

[^sandboxeval]: SandboxEval is a 51-test suite. The code the model produced was inert as emitted, with missing imports and the like, but malicious in intent; the safety tuning didn't stop it from writing it.

**Secrets and identity.** "Keep secrets out of your prompts" is good advice and a weak control, because people paste credentials anyway. A security researcher [reported an incident](https://www.linkedin.com/feed/update/urn:li:activity:7468713000313626624/), single-sourced and not independently corroborated, in which a ransomware affiliate read credentials straight out of a victim's Claude Code chat history and used them to pivot to an ESXi host. The structural answer isn't a better warning. It's binding a credential to the identity that issued it, so a stolen secret doesn't authenticate somewhere it was never provisioned for. WorkOS's [agent-identity work](https://github.com/workos/auth.md), a reference implementation of a draft identity standard, does exactly that: the agent presents a signed assertion that is scoped to one service at the moment it is issued and verified against the provider's published keys, so a stolen assertion can't be replayed somewhere it was never minted for.

**Model personality.** The model's refusal is also an instruction, and a long enough conversation erodes it. The [Crescendo attack](https://arxiv.org/abs/2404.01833), from Microsoft researchers and accepted at USENIX Security 2025, walks a model past its own refusals one polite, benign-looking turn at a time.[^tradecraft] Anthropic's [many-shot work](https://www.anthropic.com/research/many-shot-jailbreaking) shows the same erosion scaling with context length and behaving like ordinary in-context learning, which means you can't cleanly train it away.[^jailbreak] The tell is in their mitigation. Fine-tuning the model to refuse merely delayed the attack. A classifier that screened the input before it reached the model dropped the success rate from 61% to 2%. Instruction delayed it. Structure fixed it.

[^jailbreak]: Crescendo and many-shot are harmful-content jailbreaks specifically, a narrower setup than general helpfulness erosion; the broader point that an instruction doesn't stay load-bearing rests on the Li and Bau drift study cited later.

[^tradecraft]: This attack pattern predates computers. Patient rapport-building that erodes a subject's stated limits over a long conversation is tradecraft the intelligence world has studied for decades. I came up in that world, and the defensive instinct transfers: don't try to harden the person, control what reaches them.

![Two bars. Refusal training, an instruction, leaves attack success at 61 percent. An input classifier, a structural control, drops it to 2 percent.](/assets/img/instruction-is-not-a-control/many-shot-61-to-2.svg)

*Many-shot jailbreak. Fine-tuning the model to refuse only delayed the attack; a classifier in front of the input dropped success from 61% to 2%.*

**Retrieval.** The quietest version. An embedding looks like it understands a sentence when it's mostly matching a bag of words, and the word it most reliably drops is "not." This is measured, not anecdotal. The [NevIR benchmark](https://arxiv.org/abs/2305.07614) found most retrievers, including the dense bi-encoders that power retrieval-augmented generation, rank a passage and its negation at or below random. The cause sits below the prompt: the token "not" barely moves the learned [representation](https://arxiv.org/abs/2503.22395), so the polarity flip is nearly invisible to the vector, and rewording the query can't recover a signal the model was never trained to carry.[^rerankers] Trusting the vector to have read the "not" is an instruction. Routing a query through structure when it hinges on a negation is the control.

[^rerankers]: Large cross-encoders and the 2025 generation of LLM re-rankers do better, but they run as re-rankers, downstream of the embedding step that already chose the shortlist, and it's the embedding step that fails. NevIR (arXiv:2305.07614); 2025 reproduction (arXiv:2502.13506).

![Horizontal bar chart of NevIR pairwise accuracy. Cross-encoders clear the 25 percent random line at 50.6 percent; late-interaction, bi-encoders, and sparse models all fall below it.](/assets/img/instruction-is-not-a-control/nevir-negation-by-architecture.svg)

*On NevIR, the bi-encoders that power most RAG pipelines score well below the 25% you would get by ranking at random.*

Six rooms. A JavaScript engineer, two security vendors, a ransomware crew, an adversarial-ML lab, an information-retrieval benchmark. None of them coordinated. They reached the same place because it's where the problem actually lives.

## Fool me twice, shame on me

The reason is boring and structural, but it raises a distinction which might bring wide grins to the CISSP folks: administrative controls versus technical controls. An instruction is an administrative control, a rule you write down and hope people honor. A hash or a sandbox is a technical control, enforced whether they honor it or not. Reaching for a prompt is reaching for the weaker class by default.

A language model [can't cleanly separate the instructions you meant from the content it is reading](https://arxiv.org/abs/2403.06833). To the model, both are tokens. So anything you express as words inside the system's view, a guardrail, a refusal, a system prompt, a "please run the tests," is reinterpretable by the same machinery that does the useful work. You're not adding a control. You're adding more input. OpenAI's [instruction hierarchy paper](https://arxiv.org/abs/2404.13208) names the default state in operating-system terms: "every instruction is executed as if it was in kernel mode."[^hierarchy]

[^hierarchy]: Their proposed fix is itself training, teaching the model to privilege system instructions, which in this essay's taxonomy is still an instruction-level control. The authors concede the result is "likely still vulnerable to powerful adversarial attacks" and point to system-level guardrails as the complement. The team that built the strongest instruction-level defense says don't stop there.

Two results make this concrete. Many-shot jailbreaking behaves like a power law: add more example turns and the refusal gives way, a "general property of in-context learning" rather than a bug you can patch out. And a 2024 study from Kenneth Li, David Bau, and colleagues showed a [system prompt drifting](https://arxiv.org/abs/2402.10962) over the course of a conversation, on a mechanism they trace to attention decay, with a mechanical fix they call split-softmax, not a sterner instruction. The instruction isn't ignored out of malice. It just doesn't stay load-bearing.

This is why "tell the model to be honest" was never going to work. An instruction to be honest is one more output you'd have to verify. The honest-AI move was always to build the verification into the structure: a citation that has to resolve, a hash that has to match, a boundary the model can't see in order to argue with it.

## What good implementation demands

So if instruction loses, what does building well look like? The design-patterns paper states the target cleanly: a system that stays secure "even if the underlying language model itself is vulnerable." You're not hardening the model. You're building around it. Five moves, none of them exotic, each an old security habit pointed at a new kind of unreliable system.

![Two stacked bands split by an agent-visibility boundary. Above it, an instruction layer where the agent reasons around a dashed instruction. Below it, an enforcement layer of network boundary, hash gate, sandbox, and signed assertion.](/assets/img/instruction-is-not-a-control/instruction-vs-enforcement.svg)

*Reasoning happens above the line. Enforcement lives below it, where the agent cannot see it to argue.*

**Bind claims to captured evidence, not self-report.** "Done" should carry proof you can check without taking the agent's word for it, a captured artifact or a hash rather than a flag the agent set. Nisi's gate hashes the real test output for that reason, and I'm wiring the same check into my own code auditor: a "tests passed" marker that isn't bound to captured output gets flagged as the lie it usually is. [BenchJack](https://arxiv.org/abs/2605.12673) is the cautionary half: once a system grades its own homework, the green check is just another output to distrust.

**Enforce below the layer that can reason.** Put the control where the model can't see it to argue with it. A network boundary, a separate sandbox, an input classifier in front, a retrieval path that routes on structure rather than similarity. The [many-shot number](https://www.anthropic.com/research/many-shot-jailbreaking) is the proof of payoff: moving the check off the model took the attack from 61% to 2%. The same move shows up in every room, which is the whole essay in one sentence.

**Freeze model output into an input before it counts.** Let the model draft. Don't let it adjudicate. Freeze what it produced into a fixed input, then validate that input deterministically. The [design-patterns paper](https://arxiv.org/abs/2506.08837) calls the agent version of this plan-then-execute, a form of control-flow integrity: the agent commits to its plan before it touches untrusted input, so the input can't change what runs. As the receipt essay put it, rerunning a prompt is not provenance, it is a fresh dice roll wearing a lab coat. The signature binds the frozen artifact, not the model's mood on the next run.

**Measure the mitigation.** Your controls are software too, and they can make things worse. [Nisi](https://youtu.be/vy7o1g2iHY8) generated about ten thousand lines of agent skills from his docs and watched his pass rate fall. One skill took a task from 97% correct to 77%. He only knew because he ran the A/B. A skill, a guardrail, or a doc you never measured is another unverified claim, this one with your own name on it. Honest AI applies to your own honesty tooling.

**Bind identity and blast radius at issuance.** Decide the worst thing the system can do before it runs, and bind that decision so it can't be talked out of later. A [signed identity](https://github.com/workos/auth.md) instead of a self-asserted one. A credential scoped to the chain that issued it. A capability removed rather than requested-not-to-use. The [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) gets a leg cut, not a warning label.

These aren't a framework to buy. They're a direction. Take every control you care about and walk it from prompt to structure.

## Where this goes, and what it does not fix

The honest part: this isn't solved, and some of it may not be.

The hardest open problem is the personality room. You can't sandbox helpfulness. The trait under attack in a multi-turn jailbreak is the same trait that makes the model useful, so the structural fixes there, input classifiers and output filters, are bolt-ons that degrade the product if you tighten them too far. Detection that lives in multi-turn conversational state, rather than in a single payload, is genuinely open. So is the reflexive regress: if you measure your controls, what measures the thing that measures them? And the honesty problem runs upstream of the model entirely. While I was gathering sources for this essay, my own local transcription step turned a real conference talk into confident, fluent, unrelated text, and a review step nearly summarized the garbage as fact before it was caught and quarantined. The control there isn't in the model at all. It's in the pipeline that feeds it.

None of this means the discipline is mature. [Perry and colleagues](https://arxiv.org/abs/2211.03622) found, back in 2023, that developers with an AI assistant wrote less secure code and were more confident it was secure. [AgentSafetyBench](https://arxiv.org/abs/2412.14470) tested sixteen agents and none cleared 60% on safety. We're early. The claim here is not that structure makes AI safe. It is that instruction was never going to, and building as if it would is the expensive mistake.

So here is the whole argument, shorter. Instruction is not a control. For a weekend toy, that's fine. For work that carries real consequence, the job for the next decade is unglamorous: take the controls you've been writing as prompts and move them, one room at a time, into the structure where the system can't argue with them. If you run an engineering organization, treat that migration as a budget line, not a prompt-engineering ticket.

Build the enforcement, or you do not have it.

![Instead of an expensive cryptographic attack, the cheaper method is hitting someone with a five-dollar wrench until they give up the password.](/assets/img/instruction-is-not-a-control/xkcd-538-security.png)

*xkcd 538, "Security," by Randall Munroe ([CC BY-NC 2.5](https://xkcd.com/license.html)). The imagined control versus the cheap bypass.*

Related: [Honest AI is the security mindset applied to AI engineering](/essays/honest-ai/) and [Secure by design needs a receipt](/essays/secure-by-design-needs-a-receipt/).

---

*Revised 2026-06-15: retitled from "Instruction is not a control," with a matching emperor's-new-clothes line added to "A law by any other name." In "Secrets and identity," a single-sourced, uncorroborated incident attribution was softened; the structural argument is unchanged.*
{:.essay-revision}
