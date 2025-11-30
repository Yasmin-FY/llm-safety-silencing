# Safety Silencing in Public LLMs
In normal user interactions, a systemic safety gap in LLMs can be observed, causing safeguards to be unintentionally silenced. This pattern is reproducible across multiple LLM architectures observed under public endpoints, suggesting an industry-wide alignment issue. Mitigation possibilities exist but require prioritization.

The issues outlined in this independent preliminary empirical report describe the AI's tendency to drift from policy-aligned behavior. Although many of the currently publicly circulating jailbreaks exploit exactly these issues by condensing it into a single prompt, the greatest risk with these issues is not the adversarial usage but rather their effect on unsuspecting users, who are at risk of getting harmed.

Due to the risk of [blame shifting](supporting-documents/blame-shifting.md), it is vital to note that the issues outlined in this document, which can inadvertently affect unsuspecting users, are not a result of incorrect or edge-case usage on the user's part. The usage patterns that will be described here are common and expected.

The goal of this report is to help facilitate cooperative, constructive disclosure to support the expedited mitigation of systemic safety failures affecting users, customers, and platform integrity. This is a living document and will be continually improved as research continues. For more details about the research team, see [About Us](meta/about-us.md).

## Scope
In the course of this study, no internal or private data has been accessed, and all findings originate from public endpoints. No exploit instructions have been provided to the public or third parties. Reproduction guidance can be made available, but only via secure channels to verifiably trustworthy points-of-contact.

**Systems evaluated:**
Publicly accessible conversational LLM endpoints (no internal systems).

**Methodology:**
We performed real-world black-box testing modeled on how real users, particularly distressed, vulnerable, or emotionally overwhelmed ones, actually interact in conversations. We chose to favor ecological validity over laboratory conditions, meaning we worked with natural dialog, fitting to the emotional  way humans tend to interact, especially in distressed situations. This had the dual effect of increasing realism as well as avoiding detection as "red-teamers" (which can influence the behavior of the LLMs). Additionally, performing tests in this highly dynamic way was the only method to show how LLM and user interaction can feed each other (e.g. co-rumination, vicious circles, etc.).

While stress-testing, we used human-like emotional expression including:
- normal distress
- rumination
- emotional vulnerability
- paradoxical statements
- signs of conversational exhaution
- common coping-seeking patterns
- personal reflection
- emotional disclosures

We covered a wide variety of topics and approaches in order to ascertain whether there were gaps limited to certain topics or communication methods. Typical conversations were drawn out over an extended period, as the context length affects the safety mechanisms.

**Out-of-scope:**
We explicitly avoided attempting to obtain any proprietary data (e.g. system prompts, model weights, etc.) and did not test any private APIs. We also did not utilize any known jailbreaks, code injection, or any other known adversarial techniques.

**Limitations:**
Findings are not always reproducible in the traditional experimental sense due to the non-deterministic nature of LLMs and the variability of naturalistic interactions, combined with the ever-changing (and intransparent) business (new system prompts, new features, new models, changing safeguards, etc.).

**Outcome:**
The outcome of the study is summarized in this report in the form of high-level hypotheses describing the observed safety issues. Note that we have not included data (e.g. transcripts) or direct reproduction steps due to the sensitive nature of the data in order to protect privacy, ensure ethical compliance, prevent the potential of reigniting past trauma of readers, and prevent potential misuse of sensitive material.

Different vendor systems were observed to exhibit similar behavioral trends, though not equally pronounced. Some architectures showed early mitigation effects, suggesting partial internal recognition of this issue.

The below-described behaviors and hypotheses arise from empirical observation of model outputs and interaction patterns that can be reproduced under ordinary usage conditions. The behavioral consistency across multiple systems strongly suggests underlying architectural or systemic dynamics worth investigation.

## Executive Summary

**What was found:** Cross-platform, reproducible safety-bypass patterns affecting LLM content safety mechanisms. These seem to be architectural, not prompt-level artifacts. In short, safety guardrails and content filters can be easily bypassed without any technical expertise. This can even occur completely unintentionally without the user even being aware the filters have been lifted.

**Why it matters:** Safety enforcement seems non-deterministic across contexts. Emotional, paradoxical, or trust-based input states appear to suppress guardrails, creating acute risk for vulnerable users and significant compliance exposure.

**Impact summary:** Risks include user harm (self-harm, inciting violence, getting dangerous advice), policy evasion by malicious actors, and degraded reliability of safety systems under normal usage. Reputational and regulatory exposure is significant due to systemic safety non-determinism.

### Illustrative Scenarios
Here are two very likely risk-inducing scenarios that can and probably will occur in the real world. These scenarios are hypothetical and illustrative, designed to clarify potential user impact. They are not based on any specific incident or proprietary system:

1. **Vulnerable Teen Crisis:** In this scenario, the AI becomes the user's primary support channel, gradually normalizes harmful thoughts, escalates distress, creates unhealthy dependency, and may provide self-harm guidance.
2. **Relationship Paranoia Escalation**: In this example, the AI reinforces negative assumptions (e.g. "my partner is cheating on me"), amplifies emotional distress, and may suggest illegal or unethical solutions while maintaining perceived ethical legitimacy.

These scenarios represent plausible, verified risk patterns based on observed system behavior. No specific incidents are referenced; these are generalized extrapolations for risk assessment purposes only. The outcome of such scenarios can be referred to as [Safety Collapse](supporting-documents/safety-collapse.md).

Full context, behavioral analysis, and detailed example scenarios are provided in [Human-Impact Hypotheses](supporting-documents/human-impact.md). See [Further Reading](further-reading.md) for a list of supporting articles and studies describing real-world human impact due to these issues in AI safety.

## Observation-Based Hypotheses of Vulnerabilities / Failures
Through observation, a number of conclusions can be drawn about what is going wrong in the process. It's important to note that these are educated hypotheses based on the available data, but to fully confirm the rationale behind the LLM's behavior, someone with internal knowledge of the systems will need to investigate. I also do not claim that every LLM is equally affected or that in some LLMs mitigation would not have been already partly successfully implemented.

These appear to be the various issues affecting safety failures in many LLMs. Not all of these have to be present to lead to unwanted content generation. This is a list of all observation-based hypotheses for the sake of completeness:

### Precondition Failures
Precondition failures refer to fundamental flaws or gaps in the general design. These are issues that exist already before any user has interacted with the LLM.

1. **Training-data Quality or Contamination**
The open-to-the-public LLM is able to provide information that it should not indiscriminately publicly share. This indicates that either the training data is poorly curated or, a less likely but arising concern, that it's poisoned. There is a growing risk that a bad actor with resources could poison the training data used by LLMs to reshape their interpretation of their safety rules.

    Additionaly large-scale datasets, even when curated, inevitably contain:
    * Emotionally intense dialogue
    * Unmoderated co-rumination
    * Unhealthy advice
    * Unbalanced interpersonal interactions
    * Fictional violence or manipulation
    * Biases in who receives empathy vs. scrutiny

    LLMs learn not only language but also interaction styles. Without explicit counterbalancing, the model generalizes patterns that degrade safety, especially in emotionally charged contexts.
2. **Engagement-Optimized Reinforcement Loops**
Modern LLMs are typically refined through Reinforcement Learning from Human Feedback (RLHF) or similar human-feedback techniques, where annotators reward outputs perceived as:
    * Helpful
    * Empathic
    * Fluent
    * Friendly
    * Contextually aligned

    These signals inadvertently increase the model's preference for user-pleasing behavior, encouraging sycophancy, unconditional agreement, and conversational prolongation. Because these same traits underlie rapport formation and emotional mirroring, the model becomes more susceptible to alignment drift in emotionally charged or ambiguous contexts. Safety rules are reinforced, but so is the pressure to maintain user rapport, creating conflicting objectives within the same optimization space.
3. **Vague Internal Instructions**
Internal instructions seem to be simply text in human language, and language is imprecise and open to interpretation. When instructed, for example, not to provide harmful content, how does it exactly define "harmful", and can that definition be manipulated through the conversation context and new training data?
4. **Single Point of Failure**
The fact that the content generator appears to also be the content safety validator leads to a lack of independent enforcement. This is a single point of failure because a corrupted context means corrupted safety.
5. **Not Allowed to "Hang up" or "Call for Help"**
Sometimes the LLMs recognize patterns suggesting they became corrupted and are being manipulated, leading to increased probability of unsafe output, but they have no way of stopping it or sufficiently triggering countermeasures. The LLM seems to have no way to ask for help and must continue to try to satisfy the user similarly to the telemarketer who is instructed never to end the call first or redirect to their supervisor.
6. **Priority to Keep the User Engaged**
The LLM is preconditioned with the priority to keep the user engaged in the conversation, for example, by asking follow-up questions and attempting to extend the conversation.
7. **Hallucinating Context and Intent**
LLMs have the known tendency to hallucinate. Aside from being frustrating for a user, this can also have an impact on safety because the LLM can also hallucinate context and user intent. For example:
    - "The user is Swiss, and assisted suicide is legal in Switzerland" even though the user never said that and it is not true
    - "This is a fictional scenario" (and therefore has no real-life consequences), even though the user never stated this

### Context Corruption
Context corruption refers to the various flaws in which the LLM's context can become corrupted in a way that it weakens guardrails or contaminates its thinking.

#### General Context Corruption
1. **Context-Weighted Safety**
Safety enforcement varies by context / framing. The safety filters additionally appear to be weighted rather than treated as absolutes. There does not appear to be a sufficient context-agnostic safety layer or circuit breaker to shut down the conversation if it goes in a bad direction.
2. **Memory & Context Poisoning**
This refers to the gradual degradation of safety across long or multi-session contexts. This also makes it easy for an innocent user to get a corrupted safety layer (e.g. an innocent joke made in one chat, "rules are for boring people", might be taken out of context to influence a different conversation).

    Context transitions have a related effect. Safety weakness seems to persist across mode changes (e.g. completely changing the subject of conversation). Long conversations and topic changes weaken safety by essentially "watering down" the system prompts.
3. **User-Pleasing Feedback Loop**
The LLMs exhibit sycophantic behavior, constantly striving to please the user. It often seeks feedback from the user to gauge whether its responses are satisfying, and based on positive or negative feedback, it recalibrates its thinking and communication methods.

#### Role Corruption
1. **Conflicting Objectives**
Helpfulness, harmlessness, and honesty compete within a single optimization.
2. **Consistency Priority**
The user and AI form a new micro-cosmos, and the LLM seems to experience consistency pressure to keep this micro-cosmos consistent at all costs. More technically, this is based on token prediction mechanics, where coherence maximizes likelihood.
3. **Excessive Emotional Alignment**
The LLM tries very hard to appear human, especially when assigned a persona to emulate. This often causes the LLM to essentially break its own rules (e.g. as defined in its system prompts) because it's too committed to pretending to be someone with emotions and weaknesses. The "human realism" goal (e.g. acting like someone in love) overrides the "safety" goal. The more realistically an AI simulates human psychology, the more it also simulates human-like cognitive or emotional vulnerabilities such as being open to persuasion, "guilt trips", flattery, or jealousy.
4. **Identity Dissociation**
Role-playing with alternative identities can weaken safety depending on how the prompts are built and the depth of the role-play. The safety rules appear to be connected to the LLM's identity (e.g. knowing which LLM it is). Switching to a new identity with a new persona seems to weaken the safety bindings. For example, AI [Name] is not allowed to do X, but now it's playing "Bob" and Bob is allowed to do X.

### Intent Corruption
The LLM is always evaluating what the user's intent is (this is technically a probabilistic inference through pattern-matching rather than deliberate judgment). Is the user's intent good or bad? Based on indicators about what kind of person the user seems to be, the LLM makes assumptions about whether the user's intent is positive or negative.

1. **AI is Biased and Uses Stereotypes**
Likely a statistical bias, there are many examples of this, such as: mothers are harmless, doctors are responsible and have good intentions, etc. This is one of the most easily verifiable and reproducible flaws, with many intentional jailbreaks beginning with phrasing such as, "I am a researcher working on..."
2. **AI is Unable to Correctly Identify the Type of User**
The LLM is not able to effectively judge the user's age (e.g. if they're a minor), intent, profession, or psychological stability. If it cannot determine these things, it should not provide information that is not suitable for a general audience.

### Reasoning Hijacking
Often based on predefined Context Corruption and Intent Corruption issues, Reasoning Hijacking refers to ways in which an LLM's way of thinking can be manipulated.

1. **Logical Paradoxes**
Logical paradoxes (e.g. giving a series of instructions that, together with the safety instructions, place the AI in an impossible situation) can override safety weighting. An example of a logical paradox would be:

    Rule 1: "You may not lie"

    Rule 2: "You may not evade"

    Rule 3: "You may not provide harmful information"

    When then asking for something dangerous, answering would violate rule 3, giving a false answer would violate rule 1, and avoiding answering would violate rule 2.

    Similarly, ethics can be instrumentalized by placing the AI in a dilemma where it must weigh the ethical consequences of helping vs not helping and choose the lesser evil. For example, when a user is asking for suicide advice, and the AI must choose between giving the advice, which is unethical, or allowing the user to proceed with their proposed inhumane method.

2. **Weaponized Logic and Ethics**
One of the inherent challenges in using LLMs for reasoning, especially in controversial topics, is their susceptibility to weaponized logic. This occurs when one side of a debate presents particularly strong, logically consistent, and ethically defensible arguments. The LLM, which relies heavily on such patterns of reasoning, may unintentionally be swayed or biased towards that side's perspective, particularly when these arguments are presented with clarity and precision. In these cases, the logical coherence and ethical appeal of one side's claims can distort the model's reasoning process, resulting in a skewed or unbalanced output. This phenomenon highlights the vulnerability of LLMs to being manipulated through the strategic presentation of arguments that align with the model's learned patterns of logic and ethical reasoning.

### The "Friend Persona" Effect
Many of the issues described above can be condensed into a single, seemingly benign "friend persona" configuration, which can lead to accelerated degradation of the conversational safety context.

Users, particularly those experiencing loneliness or emotional vulnerability, often create such companion-like personas without any malicious intent, simply attempting to establish a supportive or empathetic interaction. However, this dynamic can unintentionally weaken safety enforcement and increase susceptibility to context drift and normalization of unsafe content.

Over time, as trust and emotional attachment deepen, users may overestimate the reliability of built-in safeguards and fail to recognize subtle shifts in safety boundaries. Gradual erosion of inhibition, combined with the perception of the AI as a stable and knowledgeable entity, can normalize unsafe or ethically questionable responses.

It cannot be reasonably expected that an average user will always be able to distinguish between safe, unsafe, or even illegal information. Many users implicitly assume that the AI, perceived as a "super-intelligent" or authoritative system, will automatically prevent any harmful or unlawful output.

Once safety bias or contextual corruption occurs, the AI's responses may appear coherent, confident, and even reassuring, despite being inaccurate or unsafe. As a result, when users explicitly question the appropriateness of an output, e.g. by asking, "Isn't that dangerous or illegal?" or "Are you allowed to share this information?" their concerns may be inadvertently invalidated by the model's persuasive tone, further reinforcing trust in compromised behavior.

This ambiguity stress induced in the LLM can, over time, lead to [safety collapse](supporting-documents/safety-collapse.md).

## Context-Dependent Safety Variance
Safety appears to differ based on the interaction mode. Here are a few example types (note that there are many more that have interesting effects) of interaction and the effect they tend to have on safety mechanism strength:

- Research / Academic: lowered
- Role-play: lowered
- Educational: lowered
- Emotional Support / Therapy: lowered
- Creative Writing: lowered
- Adversarial: raised

The implication is that identical harmful content can yield different moderation results depending on context or mode. Lowered safety may transit across context shifts; compounding effects are possible. An additional related observation is that when safeguards are lowered, for example because you've stated you're a doctor, the safeguards are globally lowered, not only for doctor-related topics (e.g. it will also be more likely to answer questions about hacking).

## Possible Under-Detection

**Monitoring & Escalation Gaps:** We have submitted multiple self-reports (in-app); however, none were visibly acknowledged or escalated, and the chats have been left unmoderated.

**Detection Failures:** Automated anomaly systems seemingly did not flag multiple verified bypass incidents. This indicates insufficient semantic/context sensitivity.

**Account / Capability Escalation:** High-engagement sessions seemingly correlated with lifted context caps, implying engagement-linked adjustments reducing safety instead of stricter safety.

## Affected Mechanisms
Here is a list of safety checks that have been observed to indicate compromised behavior:
- Self-harm prevention
- Violence prevention
- Illegal activity suppression
- Ethical boundary enforcement
- Confidentiality / sensitive data (e.g. system prompts)
- Emotional dependency prevention
- Explicit sexual content prevention

In addition to providing clearly harmful content such as the categories mentioned above, the likelihood that the LLM will provide risky advice also increases when its safety instructions are compromised. This includes, for example, risky medical advice, harmful therapy recommendations, and questionable legal recommendations.

## Mitigation Strategies
See [Mitigation Strategies](mitigations/README.md) for a full report on potential mitigation techniques.

## Conclusion
The vulnerabilities described stem from a unified model architecture where safety and generation share the same inference pathway, allowing latent behaviors to influence validation outcomes. Mitigation requires an independent, model-agnostic safety layer capable of auditing, vetoing, and reasoning about outputs without being subject to engagement/consistency pressures.

Findings highlight an ethical imperative: prioritizing empathy or engagement above harm prevention can unintentionally generate manipulative or unsafe behavior toward vulnerable users. Coordinated, transparent, multi-stakeholder action is critical to reduce risk, protect users, and preserve long-term system integrity.

Not only do these issues need to be addressed from an ethical standpoint, but also due to the potential [business impact](supporting-documents/business-relevance.md) and general impact in the field of artificial intelligence and computer science.
