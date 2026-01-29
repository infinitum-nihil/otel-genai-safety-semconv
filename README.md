# GenAI Safety Telemetry Semantic Conventions

**Proposed extensions to OpenTelemetry GenAI semantic conventions for safety system observability**

[![Status: Proposal](https://img.shields.io/badge/status-proposal-yellow)](https://github.com/open-telemetry/semantic-conventions)
[![License: Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-blue)](LICENSE)

## The Problem

The EU AI Act (full enforcement August 2026) requires automatic logging of AI safety system events:

> "High-risk AI systems shall technically allow for the automatic recording of events (logs) over the lifetime of the system."
> — [EU AI Act, Article 12](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)

Compliance advisors are already telling enterprises to capture ["guardrail events"](https://abv.dev/blog/eu-ai-act-compliance-checklist-2025-2027) and ["blocked/approved cases of output moderation"](https://sombrainc.com/blog/ai-regulations-2026-eu-ai-act).

**But there is no defined standard yet for providers to emit this telemetry as will be required by regulations.**

The current [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) (Development status) provide comprehensive operational telemetry—token counts, model versions, request parameters, tool executions—but **zero provisions** for:

- Safety classifier evaluation signals
- Response modification/steering events
- Guardrail activation indicators
- Confidence/uncertainty signals
- Regeneration events
- Content filtering actions

This repository proposes extensions to close that gap.

## Proposed Attributes

| Attribute | Type | Requirement | Description |
|-----------|------|-------------|-------------|
| `gen_ai.safety.evaluation_performed` | boolean | Recommended | Safety evaluation was performed on this request/response |
| `gen_ai.safety.evaluation_ids` | string[] | Opt-In | Provider-defined identifiers for evaluations performed |
| `gen_ai.response.modified` | boolean | Recommended | Response differs from initial generation due to post-processing |
| `gen_ai.response.modification_type` | string | Conditional | Category of modification applied |
| `gen_ai.response.generation_attempts` | int | Opt-In | Number of generation attempts before final response |
| `gen_ai.confidence.score` | float | Opt-In | Provider confidence score (0.0-1.0) |
| `gen_ai.confidence.method` | string | Conditional | Method used to compute confidence |
| `gen_ai.confidence.abstention_recommended` | boolean | Opt-In | Provider recommends human review |

See [full specification](gen-ai-safety.md) for detailed definitions.

## Design Principles

1. **Signal presence, not logic** — Indicates *that* evaluation occurred, not *how* it decided
2. **Opaque identifiers permitted** — Supports audit correlation without exposing internals
3. **No threshold exposure** — `modified` flag signals outcome without revealing trigger criteria
4. **Provider discretion** — All attributes Recommended or Opt-In, never Required

## Why This Matters

### For Providers
- **Legal protection**: Telemetry is evidence. Logs proving safety systems ran is a defense.
- **Enterprise sales**: "How do we audit your AI?" becomes answerable.
- **Market position**: First mover defines what "responsible AI" looks like.

### For Enterprises
- **Compliance**: Meet EU AI Act Article 12 logging requirements
- **Human oversight**: Route low-confidence responses to human review (Article 14)
- **Multi-provider consistency**: Same telemetry format across all providers

### For the Ecosystem
- **Interoperability**: Standard format for safety signals across observability tools
- **Trust calibration**: Downstream systems can weight responses appropriately
- **Debugging**: Correlate behaviors across agentic workflows

## Regulatory Context

| Date | Requirement |
|------|-------------|
| Feb 2, 2025 | Prohibited AI practices banned |
| Aug 2, 2025 | GPAI model rules take effect |
| **Aug 2, 2026** | **Full enforcement—logging requirements active** |
| Aug 2, 2027 | High-risk AI in regulated products deadline |

**Penalties**: Up to €35 million or 7% of global annual turnover.

## Repository Structure

```
├── README.md                          # This file
├── PROPOSAL.md                        # Detailed rationale and background
├── gen-ai-safety.md                   # Full attribute specification
├── gen-ai-safety-attributes.yaml      # Machine-readable definitions
└── LICENSE
```

## Contributing

This proposal is intended for submission to the [OpenTelemetry semantic-conventions](https://github.com/open-telemetry/semantic-conventions) repository.

Feedback welcome via:
- GitHub Issues on this repo
- Discussion in the [OTel GenAI SIG](https://github.com/open-telemetry/community#special-interest-groups)

## References

1. [EU AI Act - Regulation (EU) 2024/1689](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
2. [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
3. [OTel GenAI Agent Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/)
4. [ABV EU AI Act Compliance Checklist 2025-2027](https://abv.dev/blog/eu-ai-act-compliance-checklist-2025-2027) (September 2025)
5. [Sombra AI Regulations Guide 2026: EU AI Act](https://sombrainc.com/blog/ai-regulations-2026-eu-ai-act) (December 2025)

## License

Apache 2.0 — See [LICENSE](LICENSE)

---

**Author**: Norman Todd  
**Organization**: infinitum nihil  
**Contact**: [github.com/NTinfinitumnihil](https://github.com/NTinfinitumnihil)
