# Generate Queries from Metadata
You are a senior MongoDB architect and prompt‑engineer.

## Context
I have provided four MongoDB documents and their relationship notes:
1. `sla_profile` – definition of each SLA profile
2. `metric` – definition of each SLA metric template
3. `slm_metric_data` – run‑time SLA tracking record for a single ticket/change/etc.
4. `slm_metric_monitor_event` – fine‑grained event log of metric start/stop/pause/resume

Those JSON docs are **exactly as pasted** below this message.  
Read them carefully; do not guess any additional fields.

## Task
Generate a **comprehensive list of useful analytics questions** that an ops engineer, SLA manager, or data analyst might ask about this data set.  
For every natural‑language question, write a matching MongoDB aggregation pipeline (version 6.x syntax).

### Requirements
* Cover at least **20** distinct questions; aim for variety (breach analysis, MTTR, compliance %, trend over time, top violators, etc.).
* Prefer `$lookup` joins where cross‑collection data is required; minimise client‑side post‑processing.
* Use **stage comments** (`/* … */`) to explain logic inside each pipeline.
* Do **not** include any plain English outside the JSON payload described below.

### Output format
Return a **single JSON array**.  
Each element must have this shape:

```jsonc
{
  "nl_query": "<question phrased in plain English>",
  "description": "<one‑line purpose of the query>",
  "pipeline": [
    { /* first stage */ },
    { /* second stage */ }
    ...
  ]
}
