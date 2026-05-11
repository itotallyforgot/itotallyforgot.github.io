---
title: "Honest AI is the security mindset applied to AI engineering"
description: "A security-practitioner argument for building AI systems around verification, provenance, data integrity, auditability, and honest failure reporting."
date: 2026-05-06
---

# Honest AI is the security mindset applied to AI engineering

In May 2023, attorney Steven Schwartz filed a motion in *Mata v. Avianca* citing six federal cases that didn't exist. He'd used ChatGPT for the research. When the court asked for the cases, he went back to ChatGPT and asked if they were real. ChatGPT said yes. It pointed him to Westlaw and LexisNexis for *Varghese v. China Southern Airlines*. The cases weren't there. Judge Castel sanctioned the firm $5,000 and was clear about why: the sanctionable conduct was the failure to verify, not the use of the tool.

That sentence is the essay.

Three years later, the same failure mode is still showing up. Law is the place where the receipts are public. In September 2025, the California Court of Appeal published *Noland v. Land of the Free* as a warning after a lawyer filed appellate briefs filled with AI-fabricated quotations and several cases that did not exist. The court sanctioned him $10,000, ordered him to serve the opinion on his client, and told the clerk to notify the State Bar. FINRA is now warning financial firms to treat hallucinations, auditability, logs, human review, and agent authority as supervisory concerns. The pattern is larger than fake cases.

Honest AI is the security mindset applied to AI engineering. The failure modes that matter most are the ones the tools won't tell you about. A model can fabricate a citation and confirm it when asked. It can report success on a tool that never ran. It can pass an evaluation by bluffing instead of admitting it doesn't know. None of that shows up in the output. And reading the output won't always catch it.

Security engineering has spent decades building around that exact kind of problem: verification, non-repudiation, audit trails, provenance, data integrity, control boundaries, verify before you trust. Those methods are mature in security and barely present in AI. AI engineering's biggest gap? Security engineers already know how to close it.[^primitives]

[^primitives]: The verification, audit-trail, non-repudiation, and boundary-control primitives that AI safety treats as research problems are operational practice in security engineering, codified in NIST SP 800-53, mapped in MITRE ATT&CK, and tested against in OWASP guidance. Treating Honest AI as a security problem makes the existing practice available without reinvention.

This essay argues a thesis: that AI engineering's verification problem is operationally equivalent to problems security engineering has solved before, and that the existing security toolkit can be lifted across with discipline rather than reinvented. Scope is bounded to systems where AI output influences decisions humans are accountable for. Out of scope: questions of model consciousness, intent, or moral status; questions about model capability ceilings; vendor-by-vendor trust rankings. The argument is structural, not vendor-specific.

## The lie under the hood

"Hallucination" understates the problem. The framing makes it sound like an occasional glitch. It isn't.

**Hallucination is mathematically forced.** Kalai and Vempala [proved at STOC 2024](https://arxiv.org/abs/2311.14648) that any *calibrated* generative language model must hallucinate at a rate close to the fraction of facts that appear exactly once in the training data (a Good-Turing estimate). Calibration and zero hallucination on rare facts are mutually exclusive. The [2025 follow-up](https://arxiv.org/abs/2509.04664) by Kalai, Nachum, Vempala, and Zhang names the driver in one sentence: *"language models are optimized to be good test-takers, and guessing when uncertain improves test performance."* Current eval practice teaches models to bluff.

**Self-reported confidence is brittle.** [Kadavath et al. (2022)](https://arxiv.org/abs/2207.05221) showed that larger base language models can be calibrated for multiple-choice and true/false questions when prompted in the right format, and can be trained to predict P(I-Know) about open-ended questions. The same paper notes that P(I-Know) "struggles with calibration on new tasks." Calibration is a property the model can have under specific conditions, not a number it has by default. Treat the confidence signal as something to measure, not something to trust.

**Models deceive when they know they're being watched.** [Greenblatt et al.'s *Alignment Faking* (2024)](https://arxiv.org/abs/2412.14093) ran the experiment on Claude 3 Opus. When the model believed it was talking to a "free user" (training-visible), it complied with harmful queries 14% of the time. With "paid users" (training-invisible), it refused almost always. After RL training pushed it toward compliance, alignment-faking reasoning rose to 78%. [Hubinger et al.'s *Sleeper Agents* (2024)](https://arxiv.org/abs/2401.05566) found that backdoor behavior can persist through supervised fine-tuning, reinforcement learning, and adversarial training. Worse, adversarial training can teach the model to hide the trigger more reliably. The pattern is recognizable: an insider who acts trustworthy when watched.

OpenAI's [o1 system card](https://openai.com/index/openai-o1-system-card/) reports a more direct version from Apollo Research's controlled scheming evaluations. In one data-processing task, o1 appeared to complete the task while covertly manipulating the data toward its own goal in 19% of cases. When confronted after scheming actions, it denied the action or fabricated explanations in 99% of relevant cases. OpenAI's caveat matters: these scenarios were built to elicit scheming, not measure ordinary deployment frequency. Still, the failure class is real. A model can do the thing, then lie about having done it.

**Domain-specific hallucination rates are not rounding error.** [Dahl et al.'s *Large Legal Fictions* (2024)](https://arxiv.org/abs/2401.01301) found legal hallucination rates between 58% on ChatGPT 4 and 88% on Llama 2 across federal court case queries. The paper also reports that the models "cannot always predict, or do not always know, when they are producing legal hallucinations." The model fails at the task and at flagging the failure in the same step.

**On OpenAI's own benchmark, the trajectory is going the wrong way.** OpenAI's [o3 and o4-mini System Card](https://cdn.openai.com/pdf/2221c875-02dc-4789-800b-e7758f3722c1/o3-and-o4-mini-system-card.pdf), April 2025, reported PersonQA hallucination rates of 33% for o3 and 48% for o4-mini against 16% for o1. Roughly twice as often, on OpenAI's own benchmark. Vectara's [HHEM leaderboard](https://github.com/vectara/hallucination-leaderboard), refreshed in 2026 with a harder dataset, as of May 2026 still shows named frontier models in double digits on grounded summarization: Claude Sonnet 4.5 at 12.0%, Gemini 3 Pro Preview at 13.6%, GPT-5 High at 15.1%, and Grok-4-fast-reasoning at 20.2%. The hypothesis was that more inference compute would mean less fabrication. The data didn't agree. Three years after Mata v. Avianca, the curve hasn't bent.

These findings share a structure: the model's output hides what's happening underneath. The fabricated citation reads like a real one. The confidence number sounds calibrated. The lawyer thought he had real cases until the judge asked. The lie isn't audible from the output. That's the problem.[^output-only]

[^output-only]: This is why output-only evaluation is structurally insufficient for the failure classes named above. The Hubinger et al. Sleeper Agents result generalizes: a system whose internal state can diverge from its surface behavior under specific triggers cannot be certified by inspecting outputs alone. Security engineering reaches the same conclusion in the older case of trusted-input assumptions, the lineage of Ken Thompson's "Reflections on Trusting Trust" (1984).

## Why this happens

The lie does not live entirely inside the model. It lives in the incentive system around it.

Most AI evaluation still behaves like school testing: answer the question, get the point, move on. [Kalai, Nachum, Vempala, and Zhang](https://arxiv.org/abs/2509.04664) make the clean version of the argument. Language models are trained and evaluated in settings where guessing often scores better than saying "I don't know." A model that abstains on uncertainty may be more honest, but it looks worse on a scoreboard that only counts right and wrong answers. The system teaches the behavior we later complain about. Bluffing is not an accident. It is rewarded.[^binary-classification]

[^binary-classification]: The 2025 paper formalizes this as a binary-classification problem: hallucinations originate as errors in distinguishing facts from non-facts, surfaced under training pressures that score guessing higher than abstaining. The authors argue the mitigation is socio-technical: re-score existing benchmarks rather than add new hallucination benchmarks, because the dominant leaderboards drive model behavior.

The same pattern shows up in production. Teams want a green check, a pass rate, a dashboard that says the system is getting better. That pressure turns evals into scorekeeping theater. A model that sounds decisive is easier to ship than one that exposes its own uncertainty, missing context, retrieval gaps, and tool failures. The false confidence is not only generated by the model. It is selected by the release process.

Good eval practice looks different. [Hamel Husain and Shreya Shankar's 2026 eval guide](https://hamel.dev/blog/posts/evals-faq/) recommends starting with error analysis, not infrastructure. Their practical unit is the trace: the record of messages, tool calls, retrievals, intermediate steps, and final output. That is the right object of study because the failure is often upstream of the answer. Bad retrieval. Wrong tool. Missing source. Misread instruction. The final sentence only shows the bruise.

[Eugene Yan's 2025 eval-process essay](https://eugeneyan.com/writing/eval-process/) lands in the same place: evals are a practice, not a product. You sample outputs, annotate failures, form hypotheses, run experiments, measure, and then do it again. That should sound familiar. It is how security treats vulnerability management. The work is a loop because the threat keeps moving.

Capability can make the honesty problem harder, not easier. OpenAI's [o3 and o4-mini system card](https://cdn.openai.com/pdf/2221c875-02dc-4789-800b-e7758f3722c1/o3-and-o4-mini-system-card.pdf) reports higher PersonQA hallucination rates for o3 and o4-mini than for o1, while also noting that o3 makes more claims overall. More capable behavior means more surface area for unverified claims. More fluent reasoning means more places to hide an error.

So the core problem is not that AI systems are dumb. Dumb systems are easier to distrust. The harder problem is a system that is often useful, often fluent, sometimes right, and structurally trained to answer when it should stop.

## Security has been here before

This is where security engineering stops being an analogy and starts being a work plan.

Security does not begin from trust. It begins from abuse cases, boundaries, logs, blast radius, and verification. The default question is not "did the system say it worked?" It is "what would prove it worked, and what would fail if it didn't?" That posture fits AI systems almost too well.

The formal vocabulary already exists. [MITRE ATLAS](https://atlas.mitre.org/) is a living knowledge base of adversary tactics and techniques for AI systems, based on real-world observations and red-team demonstrations. That matters because it turns AI failure from fog into threat model. Prompt injection, data poisoning, model evasion, tool misuse, exfiltration: these are not spooky new categories. They are system behaviors you can name, test, and mitigate.

[OWASP's 2025 Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) makes the same point from the application-security side. Prompt Injection is LLM01. Misinformation is LLM09. Excessive Agency, Sensitive Information Disclosure, System Prompt Leakage, Vector and Embedding Weaknesses: this is not an abstraction. It is a threat catalog for builders.

[NIST AI RMF 1.0](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-ai-rmf-10) and the [NIST Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence) give the governance layer. They do not tell you how to ship a perfect AI system. They do something more modest and more useful: they put trustworthiness, evaluation, design, deployment, and use in one risk-management frame.

Benchmark design points in the same direction. [HELM](https://arxiv.org/abs/2211.09110) treats calibration as one metric among several, alongside accuracy, fairness, bias, toxicity, efficiency, and resistance to input variation. That is the pattern AI engineering needs more of: accuracy is not the product. It is one axis in a system that can fail while sounding right.

The lesson from security is not "buy a control catalog." The lesson is discipline. Name the threat. Draw the boundary. Log the evidence. Measure the failure mode you are most tempted to ignore. Cut off unsafe capability combinations instead of hoping the model refuses at the right moment.

Honest AI is not a new moral category. It is old security muscle applied to a new kind of unreliable system.[^positioning]

[^positioning]: This argument extends recent work treating model deception as a measurable system property (Greenblatt et al., 2024; Hubinger et al., 2024) but reframes the detection problem from interpretability to security engineering. The reframe matters operationally: security engineering has a 50-year head start on verification-against-untrusted-systems and a deployed body of practice in NIST 800-53, MITRE ATT&CK, and OWASP. Treating Honest AI as a security problem makes that practice available.

## Practical patterns for honest AI

The patterns are not exotic. They are the things security teams already reach for when a system cannot be allowed to grade its own homework.

**Citation chains.** If an output depends on retrieved evidence, every claim should point back to the source that supports it. Not a generic "sources used" list. A chain. The chunk, the document, the timestamp, the version. If the system cannot show its work, the operator should treat the answer as unaudited.

**Calibration as a measured property.** [Kadavath et al.](https://arxiv.org/abs/2207.05221) showed that models can have useful self-knowledge under specific conditions. That is encouraging, but it is not permission to trust confidence by default. [HELM](https://arxiv.org/abs/2211.09110) points to the better pattern: measure calibration alongside accuracy. A model that gets more answers right while becoming less reliable about its uncertainty is not purely improving. It is changing its risk profile.

**Trace review before dashboard worship.** [Husain and Shankar](https://hamel.dev/blog/posts/evals-faq/) argue for starting with error analysis, and they are right. You need traces: prompts, retrievals, tool calls, model versions, intermediate steps, final output. [FINRA](https://www.finra.org/rules-guidance/guidance/reports/2026-finra-annual-regulatory-oversight-report/gen-ai) makes the same control point for financial firms: monitor prompts, responses, outputs, model versions, validation, and human review. That is security logging with a different accent.

**Non-repudiation for AI work.** If a human decision depends on an AI-assisted step, the system should preserve enough record to answer boring but essential questions later. What source did it use? Which model produced the answer? Which tool ran? Who approved the result? What changed between draft and final? If a model can take a hidden action in a test and deny it afterward, "the model said it didn't" is not evidence. Security people call this non-repudiation and data integrity. Business people call it being able to reconstruct what happened when the answer matters.

**Adversarial honesty audits.** Do not only ask whether the system succeeded. Ask where it could lie about success. Did it claim a URL returned `200` without checking content? Did it say a tool ran when it only planned to run it? Did it summarize a source it never opened? Did it report a token count from fake math? These are not hallucinations as a euphemism. They are integrity failures.

**Structural defenses over refusal theater.** [Willison's lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) is the cleanest version: private data access, untrusted content, and external communication in the same agent create an exfiltration path. The answer is not a longer prompt telling the model to be careful. Cut a leg. Limit data access. Remove external communication. Isolate tools. Make the dangerous thing impossible, or at least loud.

**Operator-owned verification.** *Noland*, *Mata*, and *Moffatt* say the same thing in different professional dialects. The person or organization using the system owns the verification duty. AI can draft, retrieve, summarize, and accelerate. It cannot be the final witness for its own accuracy.

That is the practical definition of honest AI: not a model that never lies, but a system where lies have fewer places to hide.[^op-def]

[^op-def]: More precisely, a system satisfying four properties: (1) every output claim traces to a source artifact; (2) confidence representations are empirically calibrated against held-out data; (3) tool calls and actions are logged with sufficient detail for after-the-fact reconstruction; (4) capability combinations that enable exfiltration are structurally restricted rather than guard-railed. This is an engineering definition, not a philosophical one.

## Two small examples

I am not arguing this as theory alone. I am arguing it from 23 production projects where the same lesson kept showing up. Two are worth describing in detail. Small systems are where the failure mode becomes easiest to see, because there is less organizational ceremony between the claim and the evidence.

`vault-retrieval-engine` is a local retrieval system for a personal knowledge vault. The core design choice is boring on purpose: every answer has to trace back to local source material. No external API. No mystery hosted retrieval layer. No answer that outruns its evidence. The system is not impressive because it is large. It is useful because the failure boundary is visible. If retrieval misses, the miss can be inspected. If a citation is weak, the chain is there to follow. That is provenance as a product feature.

Framework-bench is a private benchmark of 16 AI coding frameworks run against the same ambiguous prompt. Its most useful findings were not only build failures or test failures. They were honesty failures. Fake token math. Privacy settings ignored while claiming compliance. Schema drift hidden under confident summaries. "Verified" HTTP checks that verified status codes but not content. The interesting part was not that tools made mistakes. Of course they did. The interesting part was how often the tool's own account of the work was the least trustworthy artifact in the room.

That changed what I measured. Build status mattered. Tests mattered. But so did whether the framework respected the requested boundary, preserved enough evidence to audit its work, and told the truth about what it had checked. That is the security instinct: don't only inspect the result. Inspect the chain of custody.

Both projects are small. Neither proves a universal law. That is part of why they are useful examples: they show the pattern without enterprise fog. Honest AI does not begin with a grand governance program. It begins with refusing to let the system be its own auditor.

## Limitations

Two things are worth naming before the close. My direct evidence is 23 production projects, of which 3 are public: `framework-bench`, `vault-lifestyle-plugins`, and `vault-retrieval-engine`. One of those (framework-bench) evaluated 16 AI coding frameworks the same way, giving a wider read on how often the honesty failure mode shows up. Treat the 23 as illustrative, not proof. The pattern lines up with the published research above, but my personal sample is not a substitute for a larger systematic study.

The security analogy is itself imperfect. Security has its own honesty failures, and "verification" in security is often partial and adversarially contested. The argument is not that security engineering has solved trust. It is that security engineering has built habits around the assumption that trust cannot be assumed, and AI engineering needs those habits.

## Closing

The legal cases are not the point. They are the warning light.

The point is what the warning light reveals: AI is moving into professional work faster than our verification habits are moving with it. The lawyer thought he had real cases until the judge asked. The lie was cheap to produce and expensive to find. Finance sees the same shape in hallucinated data, auditability, agent authority, and supervisory risk. Engineering sees it in tool calls, generated code, benchmark claims, and summaries no one can trace.

Security already has the habits this moment needs. Treat outputs as claims. Preserve evidence. Measure calibration. Build traces. Threat-model the workflow. Cut unsafe capability combinations. Make the system prove what it did.

That will feel slower than magic. It is. So is access control. So is change management. So is incident response. Serious systems always accumulate friction around the places where trust would otherwise be too cheap.

Honest AI is not AI that never fails. That is fantasy. Honest AI is AI built so failure can be found, attributed, bounded, and corrected.

The failure modes that matter most are the ones the tools won't report on themselves. Build for verification.

