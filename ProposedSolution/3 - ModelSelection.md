## Model Selection — LeMUR with `claude-3.5-sonnet` for Fast Go-To-Market


### Decision and Rationale
We will use AssemblyAI LeMUR to apply LLMs to transcripts for summaries, Q&A, action items, and report text. This choice minimizes integration work because transcription already runs on AssemblyAI, and LeMUR consumes those transcript IDs directly. With all core capabilities in-house (adapter, merge/store, DomainNER, BizLogic, Reporter), LeMUR is the quickest path to value without distracting the team with LLM hosting and prompt ops. Reference: [LeMUR – Apply LLMs to audio files](https://www.assemblyai.com/docs/lemur/apply-llms-to-audio-files).

### Scope
Post-processing language tasks (summaries, insights, structured extraction) are handled by LeMUR. We defer custom fine-tuning or self-hosted LLMs.

### Initial Configuration
Transcripts flow from Store via transcript IDs. We use task-specific prompt templates (summary, action items, insights), and select an underlying LLM using `final_model` based on cost/latency targets. Respect published rate limits (30 RPM) or request increases. 
After careful analysis, we have decided to use Claude 3.5 Sonnet as the model for LeMUR in order to extract information about the call.

### SLOs and Quality
Expect bounded, predictable latency per LeMUR task and human-reviewable output quality. 

### Risks and Mitigations
Hallucination risk is mitigated by grounding prompts strictly to transcript content and validating JSON schemas for structured outputs (where needed). Rate limits and cost are controlled with batching, caching, and token caps. PII removal will have been handled by the Async STT, as has topic extraction and entity identification, so tokens can be used for actual BI tasks. 

### Integration and Validation
The integration is straightforward: Store → LeMUR → Reporter. We will validate on a representative call set for summary fidelity, actionability, and latency; failure to meet SLOs triggers prompt/model adjustments rather than architectural changes. 
A constant feedback loop around generated tasks and insights will highlight tweaks that need to be made to prompts/models or more specific temperature/top_k configurations.

### Cost Note
LeMUR’s convenience may cost more per task than self-hosting, but reduces time-to-value and delivery risk. Very importantly, it allows us to validate hypotheses and pivot much faster and ultimately, cheaper, than by building an in-house, much more customizable solution.

#### Cost Estimate
> [!IMPORTANT]
> A few assumptions have been made for this cost estimate:
>- the average size of a medium call centre is about 100 people;
>- starting with a single call centre and expanding out sounds like a natural way to add scale organically
>- over the course of a month, your average agent will take around 37 calls/day, for a duration of ~6 minutes each. Assuming a 20-working-day month, that comes out to around 4,440 minutes/month. 
>- Marking that up to 100 agents, comes up to ~440,000 minutes or ~7,400 hours of content that needs to be filtered.


##### STT
The table below summarizes the rough STT-related costs using 7,400 hours as volume.

| Component | Volume (hrs) | Unit cost ($/hr) | Subtotal ($) |
|---|---:|---:|---:|
| Pre-recorded STT | 7,400 | 0.27 | 2,000 |
| Live Streaming STT (incl. keyterm boosting) | 7,400 | 0.19 | 1,400 |
| Entity Detection | 7,400 | 0.08 | 592 |
| Topic Detection | 7,400 | 0.15 | 1,110 |
| Key Phrases | 7,400 | 0.01 | 74 |
| PII Redaction | 7,400 | 0.08 | 592 |
| Sentiment Analysis | 7,400 | 0.02 | 148 |
| Total (approx.) | — | — | ~6,000 |

##### LeMUR
Assuming `claude3.5-sonnet` and token volumes in the ranges below:

| Item | Tokens (millions) | Unit price ($/1k tokens) | Est. monthly ($) |
|---|---:|---:|---:|
| Input tokens | 60–70 | 0.003 | 180–210 |
| Output tokens | 60–70 | 0.015 | 900–1,050 |
| Monthly total | — | — | 1,080–1,260 |

Notes: costs vary with token volume and model choice (`haiku` cheaper, `opus` higher). We’ll refine after PoV measurements.

At a cost of under $7,500/month for a 100-agent call centre making calls around the clock, this solution is cheaper than the overhead on a single L2/L3 AI Engineer. Recruitment can follow the success of the PoV where further improvements can be introduced, and some services can be brought in-house for further control.