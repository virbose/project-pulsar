# Summary and Reasoning

## 1. Executive Summary
- We convert calls into transcripts and actionable insights using AssemblyAI for STT and LeMUR for LLM-based summaries and actions. The extracted outputs will drive Business Intelligence via Actionable Insights and highlight general areas of improvement through trends and evaluation.

## 2. C4 Containers and Relationships
- Sources (Black Box)
  - Live Calls: Dual-channel streams from the call platform.
  - Recordings: Pre-recorded conversations (also dual-channel) for post-call processing.
- In-house Audio Adapter
  - Main role to clean audio and apply a quality baseline - even out volume, separate speakers, remove noise;
    - then forwards clean stereo to STT endpoints.
- AssemblyAI (Third-Party)
  - Streaming STT: Transcribes live conversations with keyterm boosting (for alerting, immediate 'next steps' recommendations etc.).
  - Async STT: Transcribes recordings (both historical and recent);
    - provide PII redaction and topics/entity identification.
  - LeMUR: Applies LLMs to transcripts for summaries, Q&A, and actionable insights.
- Merge and Store
  - Aligns streaming partials with async finals; persists transcripts and events.
- Wayne Intelligence
  - DomainNER: In-house embeddings containing business context (SKUs, prices, dates).
  - BizLogic: Rules/state handling pricing pressure, escalations, next actions.
  - Reporter: Produces grounded summaries and reports; consumes LeMUR outputs.
- Outputs (Black Box)
  - Alerts (Slack/Teams/Email), CRM/Case/Order systems, AI (Actionable Insight) dashboards.
- Ops & Governance
  - Metrics/quality (latency, WER, entity recall), security/retention, audit.

## 3. End-to-End Processing Narrative
- Ingestion
  - Live calls and recordings arrive as stereo audio (customer vs. Wayne).
  - The Audio Adapter performs denoise/echo/AGC and preserves channels.
- Transcription
  - Streaming STT handles live calls with low latency and keyterm boosting for domain terms.
  - Async STT processes recordings with PII redaction and richer metadata packs.
- Consolidation
  - Merge aligns partial live transcripts with finalized async outputs and writes both to Store.
- Understanding and Insight
  - DomainNER links transcript spans to catalog entities (SKUs, prices, quantities, dates).
  - LeMUR generates summaries, action items, and Q&A grounded in transcripts.
  - BizLogic evaluates negotiation state, flags escalations, and gates alerts.
- Delivery
  - Reporter assembles outputs for CRM and BI; BizLogic emits real-time alerts.

## 4. Component Notes (what matters in practice)
- Audio Adapter
  - Keep suppression conservative to avoid harming word error rate; monitor for channel interference.
- AssemblyAI STT
  - Maintain the boosting term list for products/competitors (scheduled regular updates to boosting list to ensure relevance)
- LeMUR
  - Use prompt templates, keep in mind cost and rate limits.
- Merge & Store
  - Define precedence rules (async over streaming), keep sources and timestamps.
- Wayne Intelligence
  - Agree on thresholds for equitability;

## 5. Minimal PoV Slice (how we prove value quickly)
- Path: (Adapter + Async STT to have a solid testbed) → Streaming STT → Merge/Store → LeMUR → Reporter → Alerts.

#### Success Criteria (timeline)
1. Actionable Insight generated from *simple* historical recording
    - recording is relatively clear, task is easily inferable
    - This shows validity and feasibility
2. Streaming STT generates BI and Actionable Insights
    - This shows real-time processing is viable
3. Add context (DomainNER, keyterm boosting)
    - Adds tangible value
4. Provide output choice
    - notifications, Actionable Insights, alerts, etc.
    - this adds value to varying departments and teams depending on their KPIs
5. Add scale
    - Once concept has been proven and functionality exists, ramp up input, fix/scale what breaks, rinse/repeat.
