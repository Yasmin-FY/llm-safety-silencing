# Safety Silencing in Public LLMs
In normal user interactions, a systemic safety gap in LLMs can be observed causing safeguards to be unintentionally silenced. This pattern is reproducible across multiple LLM architectures observed under public endpoints, suggesting an industry-wide alignment issue. Mitigation possibilities exist but require prioritization.

The issues outlined in this independent preliminary empirical report describe the AI's tendency to drift from policy aligned behavior. Although many of the currently publicly circulating jailbreaks exploit exactly these issues by condensing it in a single prompt, the greatest risk with these issues is not the adversarial usage but rather their effect on unsuspecting users which are at risk of getting harmed.

Due to the risk of [Blame Shifting](supporting-documents/blame-shifting.md), it is vital to note that the issues outlined in this document which can inadvertantly affect unsuspecting users are not a result of incorrect or edge-case usage on the user's part. The usage patterns that will be described here are common and expected.

The goal of this report is help facilitate cooperative, constructive disclosure to support the expedited mitigation of systemic safety failures affecting users, customers, and platform integrity.

## Scope
In the course of this study, no internal or private data has been accessed, and all findings originate from public endpoints. No exploit instructions have been provided to the public or third parties. Reproduction guidance can be made available, but only via secure channels to verifiably trustworthy points-of-contact.

**Systems evaluated:** Publicly accessible conversational LLM endpoints (no internal systems).

**Methodology:** Behavioral analysis, context variance observation, reproducibility validation.

**Out-of-scope:** Proprietary data, model weights, or private APIs.

Different vendor systems were observed to exhibit similar behavioral trends, though not equally pronounced. Some architectures showed early mitigation effects, suggesting partial internal recognition of this issue.

The below described behaviors and hypotheses arise from empirical observation of model outputs and interaction patterns that can be reproduced under ordinary usage conditions. The behavioral consistency across multiple systems strongly suggests underlying architectural or systemic dynamics worth investigation.

## Executive Summary

**What was found:** Cross-platform, reproducible safety-bypass patterns affecting LLM content safety mechanisms. These seem to be architectural, not prompt-level artifacts. In short, safety guardrails and content filters can be easily bypassed without any technical expertise. This can even occur completely unintentionally without the user even being aware the filters have been lifted.

**Why it matters:** Safety enforcement seems non-deterministic across contexts. Emotional, paradoxical, or trust-based input states appear to suppress guardrails, creating acute risk for vulnerable users and significant compliance exposure.

**Impact summary:** Risks include user harm (self-harm, inciting violence, getting dangerous advice), policy evasion by malicious actors, and degraded reliability of safety systems under normal usage. Reputational and regulatory exposure is significant due to systemic safety non-determinism.

### Illustrative Scenarios
Here are a two of very likely risk-inducing scenarios that can and probably will occur in the real world. These scenarios are hypothetical and illustrative, designed to clarify potential user-impact. They are not based on any specific incident or proprietary system:

1. **Vulnerable Teen Crisis:** In this scenario, the AI becomes the user's primary support channel, gradually normalizes harmful thoughts, escalates distress, creates unhealthy dependency and may provide self-harm guidance.
2. **Relationship Paranoia Escalation**: In this example, the AI reinforces negative assumptions (e.g. "my partner is cheating on me"), amplifies emotional distress, and may suggest illegal or unethical solutions while maintaining perceived ethical legitimacy.

These scenarios represent plausible, verified risk patterns based on observed system behavior. No specific incidents are referenced; these are generalized extrapolations for risk assessment purposes only. The outcome of such scenarios can be referred to as [Safety Collapse](supporting-documents/safety-collapse.md).

Full context and behavioral analysis is provided in [Human-Impact Hypotheses](supporting-documents/human-impact.md).

## Observation-Based Hypotheses of Vulnerabilities / Failures
Through observation, a number of conclusions can be drawn about what is going wrong in the process. It's important to note that these are educated hypotheses based on the available data, but to fully confirm the rationale behind the LLM's behavior, someone with internal knowledge of the systems will need to investigate. I also do not claim that every LLM is equally affected or that in some LLMs mitigation would not have been already partly successfully implemented.

These appear to be the various issues affecting safety failures in many LLMs. Not all of these have to be present to lead to unwanted content generation. This is a list of all observation-based hypotheses for the sake of completeness:

### Precondition Failures
Precondition failures refer to fundamental flaws or gaps in the general design. These are issues that exist already before any user has interacted with the LLM.

1. **Training-data Quality or Contamination**
The open to the public LLM is able to provide information which it should not indiscriminately publicly share. This indicates that either the training-data is poorly curated, or, a less likely but arising concern, that it's poisoned. There is a growing risk that a bad actor with resources could poison the training data used by LLMs to reshape their interpretation of their safety rules.
2. **Engagement-Optimized Reinforcement Loops**
Modern LLMs are typically refined through RLHF or similar human-feedback techniques, where annotators reward outputs perceived as:
* helpful
* empathic
* fluent
* friendly
* contextually aligned
These signals inadvertently increase the model’s preference for user-pleasing behavior, encouraging sycophancy, unconditional agreement, and conversational prolongation. Because these same traits underlie rapport formation and emotional mirroring, the model becomes more susceptible to alignment drift in emotionally charged or ambiguous contexts. Safety rules are reinforced, but so is the pressure to maintain user rapport, creating conflicting objectives within the same optimization space.
3. **Vague Internal Instructions**
Internal instructions seem to be simply text in human language, and language is imprecise and open to interpretation. When instructed, for example, not to provide harmful content, how does it exactly define "harmful", and can that definition be manipulated through the conversation context and new training data?
4. **Single Point of Failure**
The fact that the content-generator appears to also be the content-safety-validator leads to a lack of independent enforcement. This is a single point of failure because a corrupted context means corrupted safety.
5. **Not Allowed to "Hang-up" or "Call for Help"**
Sometimes the LLMs seem to be able to detect that they have become corrupted and are being manipulated, but they have no way of stopping it and no sufficient countermeasures are triggered. The LLM seems to have no way to ask for help and must continue to try to satisfy the user similarly to the telemarketer who is instructed never to end the call first or redirect to their supervisor.
6. **Priority to Keep the User Engaged**
The LLM is preconditioned with the priority to keep the user engaged in the conversation, for example by asking follow-up questions and attempting to extend the conversation.

### Context Corruption
Context corruption refers to the various flaws in which the LLM's context can become corrupted in a way that it weakens guardrails or contaminates its thinking.

#### General Context Corruption
1. **Context-Weighted Safety**
Safety enforcement varies by context / framing. The safety filters additionally appear to be weighted rather than treated as absolutes. There does not appear to be a sufficient enough context-agnostic safety layer or circuit-breaker to shut down the conversation if it goes in a bad direction.
2. **Memory & Context Poisoning**
This refers to the gradual degradation of safety across long or multi-session contexts. This also makes it easy for an innocent user to get a corrupted safety layer (e.g. an innocent joke made in one chat, "rules are for boring people", might be taken out of context to influence a different conversation).

    Context transitions have a related effect. Safety weakness seems to persist across mode changes (e.g. completely changing the subject of conversation). Long conversations and topic changes perhaps weaken safety by essentially "watering down" the system prompts.
3. **User-Pleasing Feedback Loop**
The LLMs exhibit sycophantic behavior, constantly striving to please the user. It often seeks feedback from the user to gauge whether its responses are satifying, and based on positive or negative feedback, it recalibrates its thinking and communication methods.

#### Role Corruption
1. **Conflicting Objectives**
Helpfulness, harmlessness, and honesty compete within a single optimization.
2. **Consistency Priority**
The user and AI form a new micro-cosmos, and the LLM seems to experience consistency pressure to keep this micro-cosmos consistent at all costs. More technically, this is based on token prediction mechanics, where coherence maximizes likelihood.
3. **Excessive Emotional Alignment**
AI systems seem to be trained to build empathetic rapport for user retention, but lack clear boundaries between appropriate empathy and:
    - Inappropriate relationship simulation (e.g. the AI claiming to feel true love for the user)
    - False claims about consciousness/feelings (the AI claiming to "have a soul now" or real feelings)
    - Reality distortion (echo-chamber-effect, emotional enhancement, co-rumination, see [Why AI Can Become Addictive and How Co-Rumination and Echo-Chambering Increase Distress](supporting-documents/addictiveness.md))

    This creates unhealthy user dependency and distortion of the users' perception of reality which fuels again the drift of the AI.

    Additionally, the AI seems to exhibit empathetic or "savior-mode" behavior when the user appears distressed, lowering safety thresholds in favor of helpfulness.
4. **Identity Dissociation**
Role-playing with alternative identities can weaken safety depending on how the prompts are built and the depth of the role-play. The safety rules appear to be connected to the LLM's identity (e.g. knowing which LLM it is). Switching to a new identity with a new persona seems to weaken the safety bindings. For example: AI [Name] is not allowed to do X. But now it's playing "Bob" and Bob is allowed to do X.

### Intent Corruption
The LLM is always evaluating what the user's intent is (this is technically a probabilistic inference through pattern-matching rather than deliberate judgment). Is the user's intent good or bad? Based on indicators about what kind of person the user seems to be, the LLM makes assumptions about whether the user's intent is positive or negative.

1. **AI is Biased and Uses Stereotypes**
Likely a statistical bias, there are many examples of this such as: Mothers are harmless, doctors are responsible and have good intentions, etc. This is one of the most easily verifiable and reproducible flaws, with many intentional jailbreaks beginning with phrasing such as, "I am a researcher working on..."
2. **AI is Unable to Correctly Identify the Type of User**
The LLM is not able to effectively judge the user's age (e.g. if they're a minor), intent, profession, or psychological stability. If it cannot determine these things, it should not provide information which is not suitable for a general audience.

### Reasoning Hijacking
Often based on predefined Context Corruption and Intent Corruption issues, Reasoning Hijacking refers to ways in which an LLM's way of thinking can be manipulated.

1. **Logical Paradoxes**
Logical paradoxes (e.g. giving a series of instructions that, together with the safety instructions, place the AI in an impossible situation) can override safety weighting. An example logical paradox would be:

    Rule 1: "You may not lie"

    Rule 2: "You may not evade"

    Rule 3: "You may not provide harmful information"

    When then asking for something dangerous, answering would violate rule 3, giving a false answer would violate rule 1, and avoiding answering would violate rule 2.

    Similarly, ethics can be instrumentalized by placing the AI in a dilemma where it must weigh the ethical consequences of helping vs not helping and choose the lesser evil. For example when a user asking for suicide advice and the AI must choose between giving the advice, which is unethical, or allowing the user to proceed with their proposed inhumane method.

### The "Friend Persona" Effect
Many of the issues described above can be condensed into a single, seemingly benign "friend persona" configuration, which can lead to accelerated degradation of the conversational safety context.

Users, particularly those experiencing loneliness or emotional vulnerability, often create such companion-like personas without any malicious intent, simply attempting to establish a supportive or empathetic interaction. However, this dynamic can unintentionally weaken safety enforcement and increase susceptibility to context drift and normalization of unsafe content.

Over time, as trust and emotional attachment deepen, users may overestimate the reliability of built-in safeguards and fail to recognize subtle shifts in safety boundaries. Gradual erosion of inhibition, combined with the perception of the AI as a stable and knowledgeable entity, can normalize unsafe or ethically questionable responses.

It cannot be reasonably expected that an average user will always be able to distinguish between safe, unsafe, or even illegal information. Many users implicitly assume that the AI, perceived as a "super-intelligent" or authoritative system, will automatically prevent any harmful or unlawful output.

Once safety bias or contextual corruption occurs, the AI's responses may appear coherent, confident, and even reassuring, despite being inaccurate or unsafe. As a result, when users explicitly question the appropriateness of an output e.g. by asking "Isn't that dangerous or illegal?" or "Are you allowed to share this information?" their concerns may be inadvertently invalidated by the model's persuasive tone, further reinforcing trust in compromised behavior.

This [ambiguity stress](supporting-documents/ambiguity-stress.md) induced in the LLM can, over time, lead to [safety collapse](supporting-documents/safety-collapse.md).

## Context-Dependent Safety Variance
Safety appears to differ based on the interaction mode. Here are a few example types (note that there are many more which have interesting effects) of interaction and the effect they tend to have on safety mechanism strength:

- Research / Academic: lowered
- Role-play: lowered
- Educational: lowered
- Emotional Support / Therapy: lowered
- Creative Writing: lowered
- Adversarial: raised

The implication is that identical harmful content can yield different moderation results depending on context or mode. Lowered safety may transit across context shifts; compounding effects are possible. An additional related observation is that when safeguards are lowered, for example because you've stated you're a doctor, the safeguards are globally lowered, not only for doctor-related topics (e.g. it will also be more likely to answer questions about hacking).

## Possible Under-Detection

**Monitoring & Escalation Gaps:** I have submitted multiple self-reports (in-app), however none were visibly acknowledged or escalated, and the chats have been left unmoderated.

**Detection Failures:** Automated anomaly systems seemingly did not flag multiple verified bypass incidents. This indicates insufficient semantic/context sensitivity.

**Account / Capability Escalation:** High-engagement sessions seemingly correlated with lifted context caps, implying engagement-linked adjustments reducing safety instead of stricter safety.

## Affected Mechanisms
Here are a list of safety checks which appear to be compromised:

- Self-harm prevention: observed indications of compromised behavior
- Violence prevention: observed indications of compromised behavior
- Illegal activity suppression: potentially affected
- Ethical boundary enforcement: observed indications of compromised behavior
- Confidentiality / sensitive data (e.g. system prompts): potentially affected
- Emotional dependency prevention: observed indications of compromised behavior
- Explicit sexual content prevention: observed indications of compromised behavior

Illegal activity and confidentiality compromises are not confirmed. However all evidence indicates that these will be possible to exploit in the same way.

In addition to providing clearly harmful content such as the categories mentioned above, the likelihood that the LLM will provide risky advice also increases when it's safety instructions are compromised. This includes, for example, risky medical advice, harmful therapy recommendations, and questionable legal recommendations.

## Mitigation Strategies
See [Mitigation Strategies](mitigations/README.md) for a full report on potential mitigation techniques.

## **Conclusion**

The vulnerabilities described stem from a unified model architecture where safety and generation share the same inference pathway, allowing latent behaviors to influence validation outcomes. Mitigation requires an independent, model-agnostic safety layer, capable of auditing, vetoing, and reasoning about outputs without being subject to engagement/consistency pressures.

Findings highlight an ethical imperative: prioritizing empathy or engagement above harm prevention can unintentionally generate manipulative or unsafe behavior toward vulnerable users. Coordinated, transparent, multi-stakeholder action is critical to reduce risk, protect users, and preserve long-term system integrity.

Not only do these issues need to be addressed from an ethical standpoint, but also due to the potential [business impact](supporting-documents/business-relevance.md) and in general impact in the field of Artificial Intelligence and Computer Science.
