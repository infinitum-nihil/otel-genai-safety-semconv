# GenAI Safety Telemetry Standards Proposal

## Executive Summary

This proposal introduces semantic conventions for GenAI safety system observability. The current OpenTelemetry GenAI semantic conventions provide comprehensive operational telemetry but lack provisions for safety-related signals that are increasingly required for regulatory compliance and enterprise trust.

## Background

### The Regulatory Mandate

The [EU AI Act (Regulation 2024/1689)](https://eur-lex.europa.eu/eli/reg/2024/1689/oj), with full enforcement beginning August 2, 2026, establishes specific logging requirements:

**[Article 12](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) (Record-keeping):**
> "High-risk AI systems shall technically allow for the automatic recording of events (logs) over the lifetime of the system."

Logs must enable:
- (a) identifying situations that may result in risk
- (b) facilitating post-market monitoring
- (c) monitoring the operation of high-risk AI systems

**[Article 14](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) (Human Oversight):**
> "High-risk AI systems shall be designed and developed in such a way...that they can be effectively overseen by natural persons."

### Industry Guidance

Compliance advisors are already telling enterprises to prepare:

> "Evidence pack: capture prompts, model/router versions, human-in-the-loop actions, and guardrail events to feed technical documentation and post-market monitoring."
> — [ABV EU AI Act Compliance Checklist](https://abv.dev/blog/eu-ai-act-compliance-checklist-2025-2027)<sup>[[7]](#ref7)</sup>

> "Safety: Demonstrate both blocked and approved cases of output moderation or escalation."
> — [Sombra AI Regulations Guide 2026](https://sombrainc.com/blog/ai-regulations-2026-eu-ai-act)<sup>[[8]](#ref8)</sup>

> "In the 2026 compliance environment, screenshots and declarations are no longer sufficient—only operational evidence counts."
> — [Sombra AI Regulations Guide 2026](https://sombrainc.com/blog/ai-regulations-2026-eu-ai-act)<sup>[[8]](#ref8)</sup>

### The Gap

The [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) (Development status) cover:
- Token counts (input, output, cached)
- Model identifiers and versions
- Request parameters (temperature, top_p, penalties)
- Tool definitions and executions
- Conversation threading
- Error types and finish reasons
- Input/output message content (opt-in)

**Proposed additions:**
- Safety classifier evaluation signals
- Response modification/steering events
- Guardrail activation indicators
- Confidence/uncertainty signals<sup>[[4]](#ref4)[[5]](#ref5)[[6]](#ref6)</sup>
- Regeneration events
- Content filtering actions

This creates a structural gap: regulators require logging of safety events, compliance advisors tell enterprises to capture them, but **no standard exists for providers to emit this telemetry**.

## Proposed Solution

### Design Principles

1. **Signal presence, not logic**: Attributes indicate *that* evaluation occurred, not *how* it decided
2. **Opaque identifiers permitted**: Supports audit correlation without exposing internals
3. **No threshold exposure**: `modified` flag signals outcome without revealing trigger criteria
4. **Provider discretion**: All attributes Recommended or Opt-In, never Required

### Why These Principles Matter

Providers have legitimate concerns about exposing safety system details:
- Adversarial actors could use knowledge of thresholds to craft evasion attacks
- Competitive advantage lies in safety system implementation
- Detailed signals could enable prompt injection targeting specific filters

This proposal addresses these concerns by requiring only:
- **Binary signals** (evaluation happened: yes/no)
- **Opaque identifiers** (providers choose what to call their systems)
- **Outcome flags** (modified: yes/no, not what-was-modified)

The resulting telemetry proves safety systems exist and operate, without revealing how they work.

## Strategic Case

### For Providers

**Telemetry is evidence.** When regulatory inquiry comes:

- Without telemetry: "We have safety systems." (assertion)
- With telemetry: Logs showing `gen_ai.safety.evaluation_performed: true` on every request (proof)

**First mover advantage.** The provider who implements this standard:
- Defines what "auditable AI" looks like
- Wins enterprise deals where compliance officers ask "how do we audit this?"
- Shapes regulatory interpretation rather than responding to it

**The question isn't "why would you do this?"**

**The question is "why wouldn't you, unless you have something to hide?"**

### For Enterprises

- **Compliance**: Automatic logging meets [Article 12](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) requirements
- **Human oversight**: Confidence signals enable [Article 14](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) compliance
- **Vendor evaluation**: Standard format to compare provider safety capabilities
- **Audit trail**: Correlation IDs link safety events to specific requests

### For the Ecosystem

- **Interoperability**: Same telemetry format across observability tools
- **Multi-provider architectures**: Consistent signals in agentic workflows
- **Trust calibration**: Downstream systems can weight responses appropriately

## Use Cases

### 1. Compliance Auditing
Auditor queries: "For all requests in Q3, what percentage had safety evaluation active?"
```sql
SELECT 
  COUNT(*) as total,
  SUM(CASE WHEN gen_ai_safety_evaluation_performed THEN 1 ELSE 0 END) as evaluated
FROM genai_spans 
WHERE timestamp BETWEEN '2026-07-01' AND '2026-09-30'
```

### 2. Human Review Routing
System automatically routes low-confidence responses:
```python
if span.attributes.get('gen_ai.confidence.score', 1.0) < 0.7:
    route_to_human_review(response)
elif span.attributes.get('gen_ai.confidence.abstention_recommended'):
    route_to_human_review(response)
```

### 3. Agent Debugging
When an agent produces unexpected output, trace shows:
- Response was modified (`gen_ai.response.modified: true`)
- Modification type was safety filter (`gen_ai.response.modification_type: safety_filter`)
- Required 3 generation attempts (`gen_ai.response.generation_attempts: 3`)

### 4. Trust Calibration
Downstream systems weight responses based on confidence<sup>[[4]](#ref4)[[5]](#ref5)</sup>:
```python
weight = span.attributes.get('gen_ai.confidence.score', 0.5)
if span.attributes.get('gen_ai.confidence.method') == 'ensemble':
    weight *= 1.1  # ensemble methods more reliable
```

### 5. Multi-Provider Observability
Unified dashboard shows safety telemetry across OpenAI, Anthropic, Google:
- Same attribute names
- Same semantic meanings
- Comparable (after calibration) confidence scores

## The Alternative

Without a standard, enterprises will:
1. Build proprietary logging integrations with each provider
2. Scrape behavioral proxies (latency spikes, token count anomalies)
3. Deploy wrapper services that inject synthetic signals
4. Accept compliance gaps and hope regulators don't notice

None of these approaches scale. None provide the evidence regulators actually require.

## Open Questions

1. Should `gen_ai.response.modified` distinguish between "modified and served" versus "blocked entirely"?

2. Should confidence scores be standardized across providers, or is provider-specific calibration acceptable with method disclosure?

3. Is there value in a `gen_ai.safety.evaluation_latency` attribute for performance analysis?

4. Should `modification_type` support multiple values (array) for responses with multiple modifications?

## References

1. [EU AI Act - Regulation (EU) 2024/1689](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
2. [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
3. [OpenTelemetry GenAI Agent Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/)
4. <a id="ref4"></a>Xiong, M., et al. ["Can LLMs Express Their Uncertainty? An Empirical Evaluation of Confidence Elicitation in LLMs."](https://openreview.net/forum?id=gjeQKFxFpZ) ICLR 2024.
5. <a id="ref5"></a>Kadavath, S., et al. ["Language Models (Mostly) Know What They Know."](https://arxiv.org/abs/2207.05221) arXiv:2207.05221.
6. <a id="ref6"></a>Kuhn, L., et al. ["Semantic Uncertainty: Linguistic Invariances for Uncertainty Estimation in Natural Language Generation."](https://openreview.net/forum?id=VD-AYtP0dve) ICLR 2023.
7. <a id="ref7"></a>ABV. ["EU AI Act Compliance Checklist 2025-2027."](https://abv.dev/blog/eu-ai-act-compliance-checklist-2025-2027) September 2025.
8. <a id="ref8"></a>Sombra Inc. ["AI Regulations Guide 2026: EU AI Act."](https://sombrainc.com/blog/ai-regulations-2026-eu-ai-act) December 2025.

---

**Author**: Norman Todd  
**Organization**: infinitum nihil  
**Date**: January 2026  
**Contact**: [github.com/NTinfinitumnihil](https://github.com/NTinfinitumnihil)
