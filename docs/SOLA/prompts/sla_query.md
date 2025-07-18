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

The filters should be dynamic using not all or them but module related
```jsonc
[ {
  "nl_query": "What is the daily SLA performance trend for {{module_id}} from {{start_date}} to {{end_date}}?",
  "description": "Show daily breach count and compliance percentage for a given module over a custom time period.",
  "collection": "slm_metric_data",
  "pipeline": [
    {
      "comment": "STEP 1: Filter for metrics in the provided date range and module_id if available.",
      "stage": {
        "$match": {
          "is_deleted": false,
          "metric_start_time": {
            "$gte":  "{{start_date}}" ,
            "$lte": "{{end_date}}" 
          },
          "module_id": "{{module_id}}"
        }
      }
    },
    {
      "comment": "STEP 2: Extract date part from metric_start_time.",
      "stage": {
        "$project": {
          "date": {
            "$dateToString": {
              "format": "%Y-%m-%d",
              "date": "$metric_start_time"
            }
          },
          "is_breached": 1
        }
      }
    },
    {
      "comment": "STEP 3: Group by date and count total vs breached.",
      "stage": {
        "$group": {
          "_id": "$date",
          "total_metrics": { "$sum": 1 },
          "breached_count": {
            "$sum": {
              "$cond": [ "$is_breached", 1, 0 ]
            }
          }
        }
      }
    },
    {
      "comment": "STEP 4: Calculate compliance percentage.",
      "stage": {
        "$project": {
          "_id": 0,
          "date": "$_id",
          "total_metrics": 1,
          "breached_count": 1,
          "compliance_percentage": {
            "$cond": [
              { "$eq": [ "$total_metrics", 0 ] },
              0,
              {
                "$multiply": [
                  {
                    "$divide": [
                      { "$subtract": [ "$total_metrics", "$breached_count" ] },
                      "$total_metrics"
                    ]
                  },
                  100
                ]
              }
            ]
          }
        }
      }
    },
    {
      "comment": "STEP 5: Sort by date ascending.",
      "stage": {
        "$sort": {
          "date": 1
        }
      }
    }
  ]
}]
```