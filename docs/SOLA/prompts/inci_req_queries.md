# Generate Queries from Metadata
You are a senior MongoDB architect and prompt‑engineer.

## Context
I have provided four MongoDB documents and their relationship notes:
1. `incident` – definition of each incident
2. `request_process` – definition of each request

Those JSON docs are **exactly as pasted** below this message.  
Read them carefully; do not guess any additional fields.

## Task
Generate a **comprehensive list of useful analytics questions** that an operation and Technician engineer, or data analyst might ask about incident and request.  
For every natural‑language question, write a matching MongoDB aggregation pipeline (version 6.x syntax).

### Requirements
* Cover at least **20** distinct questions; aim for variety (Search / Retrieval, Time-based Questions, Aggregation / Summary, Field Value Inspection, etc.).
* Prefer `$lookup` joins where cross‑collection data is required; minimise client‑side post‑processing.
* Use **stage comments** (`/* … */`) to explain logic inside each pipeline.
* Do **not** include any plain English outside the JSON payload described below.

### Output format
Return a **single JSON array**.  
Each element must have this shape:

The filters should be dynamic using not all or them but module related
```json
[
  {
    "nl_query": "How many incidents were created per day in the last 30 days?",
    "description": "Count of incidents created per day in the last 30 days.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents created in the last 30 days.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "day",
                  "amount": 30
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by creation date (YYYY-MM-DD) and count.",
        "stage": {
          "$group": {
            "_id": {
              "$dateToString": { "format": "%Y-%m-%d", "date": "$creation_time" }
            },
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by date ascending.",
        "stage": { "$sort": { "_id": 1 } }
      }
    ]
  },
  {
    "nl_query": "Top 5 impacted services by ticket volume?",
    "description": "Find the top 5 impacted services by number of incidents.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter active incidents.",
        "stage": { "$match": { "is_deleted": false } }
      },
      {
        "comment": "STEP 2: Group by impact_service_name and count.",
        "stage": {
          "$group": {
            "_id": "$basic_info.impact_service_name",
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending and limit to top 5.",
        "stage": { "$sort": { "count": -1 } }
      },
      {
        "comment": "STEP 4: Limit to top 5 services.",
        "stage": { "$limit": 5 }
      }
    ]
  },
  {
    "nl_query": "What is the weekly trend of incident creation vs resolution?",
    "description": "Weekly counts of incident creation and resolution for the last 12 weeks.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents created or resolved in last 12 weeks.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "$or": [
              {
                "creation_time": {
                  "$gte": {
                    "$dateSubtract": {
                      "startDate": "{{dt_now}}",
                      "unit": "week",
                      "amount": 12
                    }
                  }
                }
              },
              {
                "inci_resolution_time": {
                  "$gte": {
                    "$dateSubtract": {
                      "startDate": "{{dt_now}}",
                      "unit": "week",
                      "amount": 12
                    }
                  }
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 2: Project week for creation and resolution.",
        "stage": {
          "$project": {
            "creation_week": {
              "$dateToString": { "format": "%Y-%U", "date": "$creation_time" }
            },
            "resolution_week": {
              "$cond": [
                { "$ifNull": ["$inci_resolution_time", false] },
                { "$dateToString": { "format": "%Y-%U", "date": "$inci_resolution_time" } },
                null
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Group by week and count creations and resolutions.",
        "stage": {
          "$group": {
            "_id": "$creation_week",
            "created_count": { "$sum": 1 },
            "resolved_count": {
              "$sum": {
                "$cond": [
                  { "$ifNull": ["$resolution_week", false] },
                  1,
                  0
                ]
              }
            }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by week ascending.",
        "stage": { "$sort": { "_id": 1 } }
      }
    ]
  },
  {
    "nl_query": "What are the most common incident summaries/tags this quarter?",
    "description": "Top incident summaries and tags for the current quarter.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents created in current quarter.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": {
                "$dateTrunc": {
                  "date": "{{dt_now}}",
                  "unit": "quarter"
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by summary and count.",
        "stage": {
          "$group": {
            "_id": "$basic_info.summary",
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending.",
        "stage": { "$sort": { "count": -1 } }
      },
      {
        "comment": "STEP 4: Limit to top 10 summaries.",
        "stage": { "$limit": 10 }
      }
    ]
  },
  {
    "nl_query": "How many incidents are currently assigned to each engineer?",
    "description": "Count of currently assigned incidents per engineer.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter active incidents with assignee.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "current_assignment_info.assignee_profile.full_name": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Group by assignee full name and count.",
        "stage": {
          "$group": {
            "_id": "$current_assignment_info.assignee_profile.full_name",
            "assigned_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending.",
        "stage": { "$sort": { "assigned_count": -1 } }
      }
    ]
  },
  {
    "nl_query": "How many tickets did {{created_by_name}} close this week?",
    "description": "Count of tickets closed by {{created_by_name}} in the current week.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents closed by {{created_by_name}} in current week.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "close_info.closed_by": "{{created_by_name}}",
            "actual_closure_date": {
              "$gte": {
                "$dateTrunc": {
                  "date": "{{dt_now}}",
                  "unit": "week"
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Count the tickets.",
        "stage": {
          "$count": "closed_count"
        }
      }
    ]
  },
  {
    "nl_query": "Who has the highest number of overdue tickets?",
    "description": "Find the user with the most overdue tickets (not closed past due date).",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents past agreed_closure_date and not closed.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "actual_closure_date": null,
            "agreed_closure_date": { "$ne": null },
            "agreed_closure_date": { "$lt": "{{dt_now}}" }
          }
        }
      },
      {
        "comment": "STEP 2: Group by assignee and count.",
        "stage": {
          "$group": {
            "_id": "$current_assignment_info.assignee_profile.full_name",
            "overdue_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by overdue count descending.",
        "stage": { "$sort": { "overdue_count": -1 } }
      },
      {
        "comment": "STEP 4: Limit to top user.",
        "stage": { "$limit": 1 }
      }
    ]
  },
{
  "nl_query": "What’s the average resolution time per group or user?",
  "description": "Average resolution time (minutes) per group and assignee.",
  "collection": "incident",
  "pipeline": [
    {
      "comment": "STEP 1: Filter incidents with resolution time.",
      "stage": {
        "$match": {
          "is_deleted": false,
          "inci_resolution_time": { "$ne": null }
        }
      }
    },
    {
      "comment": "STEP 2: Project group, assignee, and resolution duration.",
      "stage": {
        "$project": {
          "group": "$current_assignment_info.group_name",
          "assignee": "$current_assignment_info.assignee_profile.full_name",
          "resolution_minutes": {
            "$divide": [
              { "$subtract": ["$inci_resolution_time", "$creation_time"] },
              60000
            ]
          }
        }
      }
    },
    {
      "comment": "STEP 3: Group by group and assignee, calculate average.",
      "stage": {
        "$group": {
          "_id": { "group": "$group", "assignee": "$assignee" },
          "avg_resolution_minutes": { "$avg": "$resolution_minutes" },
          "ticket_count": { "$sum": 1 }
        }
      }
    },
    {
      "comment": "STEP 4: Flatten output fields.",
      "stage": {
        "$project": {
          "_id": 0,
          "group": "$_id.group",
          "assignee": "$_id.assignee",
          "avg_resolution_minutes": 1,
          "ticket_count": 1
        }
      }
    }
  ]
},
{
  "nl_query": "Which incidents are stuck in ‘In Progress’ state for more than 2 days?",
  "description": "List incidents in 'In Progress' state for over 2 days.",
  "collection": "incident",
  "pipeline": [
      {
        "comment": "STEP 1: Filter incidents in 'In Progress' state for >2 days.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.state.name": "In Progress",
            "state_change_time": {
              "$lte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "day",
                  "amount": 2
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "state_change_time": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show all open or reassigned incidents in the last 7 days.",
    "description": "List incidents opened or reassigned in the last 7 days.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents created or reassigned in last 7 days and not closed.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "actual_closure_date": null,
            "$or": [
              {
                "creation_time": {
                  "$gte": {
                    "$dateSubtract": {
                      "startDate": "{{dt_now}}",
                      "unit": "day",
                      "amount": 7
                    }
                  }
                }
              },
              {
                "state_change_time": {
                  "$gte": {
                    "$dateSubtract": {
                      "startDate": "{{dt_now}}",
                      "unit": "day",
                      "amount": 7
                    }
                  }
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "creation_time": 1,
            "state_change_time": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "What is the current state distribution of all active tickets?",
    "description": "Distribution of active tickets by state.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter active (not closed) incidents.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "actual_closure_date": null
          }
        }
      },
      {
        "comment": "STEP 2: Group by state name and count.",
        "stage": {
          "$group": {
            "_id": "$basic_info.state.name",
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending.",
        "stage": { "$sort": { "count": -1 } }
      }
    ]
  },
  {
    "nl_query": "Which services have recurring incidents?",
    "description": "Find services with multiple incidents in last 90 days.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents created in last 90 days.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "day",
                  "amount": 90
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by impact_service_name and count.",
        "stage": {
          "$group": {
            "_id": "$basic_info.impact_service_name",
            "incident_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Filter services with more than 1 incident.",
        "stage": {
          "$match": { "incident_count": { "$gt": 1 } }
        }
      }
    ]
  },
  {
    "nl_query": "What percentage of tickets were closed without a root cause?",
    "description": "Percentage of closed tickets with empty closure_note.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter closed incidents.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "actual_closure_date": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Group for total and without root cause.",
        "stage": {
          "$group": {
            "_id": null,
            "total_closed": { "$sum": 1 },
            "no_root_cause": {
              "$sum": {
                "$cond": [
                  { "$or": [
                    { "$eq": ["$close_info.closure_note", null] },
                    { "$eq": ["$close_info.closure_note", ""] }
                  ] },
                  1,
                  0
                ]
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Project percentage.",
        "stage": {
          "$project": {
            "_id": 0,
            "percentage_no_root_cause": {
              "$multiply": [
                { "$divide": ["$no_root_cause", "$total_closed"] },
                100
              ]
            },
            "total_closed": 1,
            "no_root_cause": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "How many incidents were marked as high impact but low severity?",
    "description": "Count of incidents with high impact and low severity.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents with high impact and low severity.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.impact.name": "High",
            "basic_info.severity.name": "Low"
          }
        }
      },
      {
        "comment": "STEP 2: Count the incidents.",
        "stage": { "$count": "high_impact_low_severity_count" }
      }
    ]
  },
  {
    "nl_query": "Which locations are reporting the most issues?",
    "description": "Top locations by incident count.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter active incidents with location.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "requester.location.location_name": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Group by location and count.",
        "stage": {
          "$group": {
            "_id": "$requester.location.location_name",
            "incident_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending.",
        "stage": { "$sort": { "incident_count": -1 } }
      },
      {
        "comment": "STEP 4: Limit to top 10 locations.",
        "stage": { "$limit": 10 }
      }
    ]
  },
  {
    "nl_query": "Who are the top requesters of incidents?",
    "description": "Top incident requesters by count.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter active incidents with requester.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "requester.full_name": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Group by requester and count.",
        "stage": {
          "$group": {
            "_id": "$requester.full_name",
            "incident_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending.",
        "stage": { "$sort": { "incident_count": -1 } }
      },
      {
        "comment": "STEP 4: Limit to top 10 requesters.",
        "stage": { "$limit": 10 }
      }
    ]
  },
  {
    "nl_query": "Which incidents were assigned but never updated?",
    "description": "Incidents assigned but last_update_time equals creation_time.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents where last_update_time equals creation_time.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "current_assignment_info.assignee_profile.full_name": { "$ne": null },
            "$expr": { "$eq": ["$last_update_time", "$creation_time"] }
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "creation_time": 1,
            "last_update_time": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get all critical-priority tickets for assignee {{assignee}}",
    "description": "Fetch all incident tickets with 'Critical' priority assigned to the user.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match tickets where priority is 'Critical' and assignee full name.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.priority.name": "Critical",
            "current_assignment_info.assignee_profile.full_name": "{{assignee}}"
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant ticket fields for output.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "basic_info.priority.name": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "creation_time": 1,
            "actual_closure_date": 1,
            "basic_info.state.name": 1,
            "basic_info.status.name": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "What is the average resolution time of incidents in last 365 days for service {{service_name}}?",
    "description": "Calculate the average resolution time (in minutes) for incidents related to a specific service in the last 365 days.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter incidents created in the last 365 days and where the impact_service_name matches.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.impact_service_name": "{{service_name}}",
            "creation_time": {
              "$gte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "day",
                  "amount": 365
                }
              }
            },
            "inci_resolution_time": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Project resolution duration in minutes.",
        "stage": {
          "$project": {
            "resolution_duration_minutes": {
              "$divide": [
                { "$subtract": [ "$inci_resolution_time", "$creation_time" ] },
                60000
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Group to calculate average resolution time.",
        "stage": {
          "$group": {
            "_id": null,
            "average_resolution_time_minutes": { "$avg": "$resolution_duration_minutes" },
            "incident_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 4: Clean final output.",
        "stage": {
          "$project": {
            "_id": 0,
            "average_resolution_time_minutes": 1,
            "incident_count": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show all open incidents and requests with their priority and assignee details",
    "description": "Get a unified view of all open incidents and requests with priority and assignment information.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match open incidents (state not 'Close').",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.state.name": { "$ne": "Close" }
          }
        }
      },
      {
        "comment": "STEP 2: Add ticket type field and project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "ticket_type": "incident",
            "basic_info.summary": 1,
            "basic_info.priority.name": 1,
            "basic_info.state.name": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "current_assignment_info.group_name": 1,
            "creation_time": 1,
            "basic_info.impact_service_name": 1
          }
        }
      },
      {
        "comment": "STEP 3: Union with open requests.",
        "stage": {
          "$unionWith": {
            "coll": "request_process",
            "pipeline": [
              {
                "$match": {
                  "is_deleted": false,
                  "basic_info.state.name": { "$ne": "Close" }
                }
              },
              {
                "$project": {
                  "_id": 0,
                  "display_id": 1,
                  "ticket_type": "request",
                  "basic_info.summary": 1,
                  "basic_info.priority.name": 1,
                  "basic_info.state.name": 1,
                  "current_assignment_info.assignee_profile.full_name": 1,
                  "current_assignment_info.group_name": 1,
                  "creation_time": 1,
                  "basic_info.impact_service_name": 1
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 4: Sort by creation time descending.",
        "stage": {
          "$sort": {
            "creation_time": -1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Count tickets by priority for organization {{impact_service}}",
    "description": "Get the count of tickets grouped by priority level for a specific impact service.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match tickets for the specific organization.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.impact_service_name": "{{impact_service}}"
          }
        }
      },
      {
        "comment": "STEP 2: Group by priority and count.",
        "stage": {
          "$group": {
            "_id": "$basic_info.priority.name",
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Union with request counts.",
        "stage": {
          "$unionWith": {
            "coll": "request_process",
            "pipeline": [
              {
                "$match": {
                  "is_deleted": false,
                  "basic_info.impact_service_name": "{{impact_service}}"
                }
              },
              {
                "$group": {
                  "_id": "$basic_info.priority.name",
                  "count": { "$sum": 1 }
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 4: Group again to sum counts from both collections.",
        "stage": {
          "$group": {
            "_id": "$_id",
            "total_count": { "$sum": "$count" }
          }
        }
      },
      {
        "comment": "STEP 5: Sort by total count descending.",
        "stage": {
          "$sort": {
            "total_count": -1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get incidents created by source {{source}} in the last {{days}} days",
    "description": "Retrieve incidents created through a specific source (Web, Email, etc.) within a date range.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Filter by source and date range.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.incident_source.name": "{{source}}",
            "creation_time": {
              "$gte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "day",
                  "amount": "{{days}}"
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields and add aging information.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "basic_info.incident_source.name": 1,
            "basic_info.priority.name": 1,
            "basic_info.state.name": 1,
            "creation_time": 1,
            "requester.full_name": 1,
            "age_days": {
              "$divide": [
                { "$subtract": [ "{{dt_now}}", "$creation_time" ] },
                86400000
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by creation time descending.",
        "stage": {
          "$sort": {
            "creation_time": -1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find all tickets with watchers containing email {{email}}",
    "description": "Search for tickets where a specific email address is in the watchers/followers list.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match tickets where the email is in current_watcher array.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "current_watcher.email": "{{email}}"
          }
        }
      },
      {
        "comment": "STEP 2: Project ticket details and filter watcher info.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "basic_info.state.name": 1,
            "basic_info.priority.name": 1,
            "creation_time": 1,
            "watchers": {
              "$filter": {
                "input": "$current_watcher",
                "cond": { "$eq": [ "$$this.email", "{{email}}" ] }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Union with requests containing the same email in watchers.",
        "stage": {
          "$unionWith": {
            "coll": "request_process",
            "pipeline": [
              {
                "$match": {
                  "is_deleted": false,
                  "current_watcher.email": "{{email}}"
                }
              },
              {
                "$project": {
                  "_id": 0,
                  "display_id": 1,
                  "basic_info.summary": 1,
                  "basic_info.state.name": 1,
                  "basic_info.priority.name": 1,
                  "creation_time": 1,
                  "watchers": {
                    "$filter": {
                      "input": "$current_watcher",
                      "cond": { "$eq": [ "$$this.email", "{{email}}" ] }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get monthly incident trends for the last 12 months",
    "description": "Generate monthly incident creation trends over the past 12 months.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match incidents from the last 12 months.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "month",
                  "amount": 12
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by year-month and count incidents.",
        "stage": {
          "$group": {
            "_id": {
              "year": { "$year": "$creation_time" },
              "month": { "$month": "$creation_time" }
            },
            "incident_count": { "$sum": 1 },
            "high_priority_count": {
              "$sum": {
                "$cond": [
                  { "$eq": [ "$basic_info.priority.name", "High" ] },
                  1,
                  0
                ]
              }
            },
            "critical_priority_count": {
              "$sum": {
                "$cond": [
                  { "$eq": [ "$basic_info.priority.name", "Critical" ] },
                  1,
                  0
                ]
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by year and month.",
        "stage": {
          "$sort": {
            "_id.year": 1,
            "_id.month": 1
          }
        }
      },
      {
        "comment": "STEP 4: Format output with month names.",
        "stage": {
          "$project": {
            "_id": 0,
            "year": "$_id.year",
            "month": "$_id.month",
            "month_name": {
              "$arrayElemAt": [
                ["", "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"],
                "$_id.month"
              ]
            },
            "incident_count": 1,
            "high_priority_count": 1,
            "critical_priority_count": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show workload distribution across teams for organization {{organization_id}}",
    "description": "Display ticket distribution and workload metrics across different teams.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match tickets for the organization.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "organization": "{{organization_id}}"
          }
        }
      },
      {
        "comment": "STEP 2: Group by team and calculate metrics.",
        "stage": {
          "$group": {
            "_id": "$current_assignment_info.group_name",
            "total_tickets": { "$sum": 1 },
            "open_tickets": {
              "$sum": {
                "$cond": [
                  { "$ne": [ "$basic_info.state.name", "Close" ] },
                  1,
                  0
                ]
              }
            },
            "high_priority_tickets": {
              "$sum": {
                "$cond": [
                  { "$eq": [ "$basic_info.priority.name", "High" ] },
                  1,
                  0
                ]
              }
            },
            "avg_resolution_time_hours": {
              "$avg": {
                "$cond": [
                  { "$ne": [ "$inci_resolution_time", null ] },
                  { "$divide": [ { "$subtract": [ "$inci_resolution_time", "$creation_time" ] }, 3600000 ] },
                  null
                ]
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Union with request workload data.",
        "stage": {
          "$unionWith": {
            "coll": "request_process",
            "pipeline": [
              {
                "$match": {
                  "is_deleted": false,
                  "organization": "{{organization_id}}"
                }
              },
              {
                "$group": {
                  "_id": "$current_assignment_info.group_name",
                  "total_tickets": { "$sum": 1 },
                  "open_tickets": {
                    "$sum": {
                      "$cond": [
                        { "$ne": [ "$basic_info.state.name", "Close" ] },
                        1,
                        0
                      ]
                    }
                  },
                  "high_priority_tickets": {
                    "$sum": {
                      "$cond": [
                        { "$eq": [ "$basic_info.priority.name", "High" ] },
                        1,
                        0
                      ]
                    }
                  },
                  "avg_resolution_time_hours": {
                    "$avg": {
                      "$cond": [
                        { "$ne": [ "$request_resolution_time", null ] },
                        { "$divide": [ { "$subtract": [ "$request_resolution_time", "$creation_time" ] }, 3600000 ] },
                        null
                      ]
                    }
                  }
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 4: Group again to sum metrics from both collections.",
        "stage": {
          "$group": {
            "_id": "$_id",
            "total_tickets": { "$sum": "$total_tickets" },
            "open_tickets": { "$sum": "$open_tickets" },
            "high_priority_tickets": { "$sum": "$high_priority_tickets" },
            "avg_resolution_time_hours": { "$avg": "$avg_resolution_time_hours" }
          }
        }
      },
      {
        "comment": "STEP 5: Sort by total tickets descending.",
        "stage": {
          "$sort": {
            "total_tickets": -1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find employee onboarding requests with status {{status}} and employee type {{employee_type}}",
    "description": "Retrieve onboarding requests filtered by status and employee type.",
    "collection": "request_process",
    "pipeline": [
      {
        "comment": "STEP 1: Match onboarding requests with specific status and employee type.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.request_type.name": "Onboarding",
            "employee_info.status.name": "{{status}}",
            "employee_info.employee_type": "{{employee_type}}"
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields including employee information.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "basic_info.state.name": 1,
            "basic_info.priority.name": 1,
            "creation_time": 1,
            "requester.full_name": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "employee_info.employee_id": 1,
            "employee_info.employee_type": 1,
            "employee_info.personal_email_id": 1,
            "employee_info.department": 1,
            "employee_info.status.name": 1,
            "template_info.template_name": 1
          }
        }
      },
      {
        "comment": "STEP 3: Sort by creation time descending.",
        "stage": {
          "$sort": {
            "creation_time": -1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show request template usage statistics for organization {{organization_id}}",
    "description": "Analyze usage patterns of request templates within an organization.",
    "collection": "request_process",
    "pipeline": [
      {
        "comment": "STEP 1: Match requests for the organization with template info.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "organization": "{{organization_id}}",
            "template_info.template_name": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Group by template and calculate usage metrics.",
        "stage": {
          "$group": {
            "_id": "$template_info.template_name",
            "total_requests": { "$sum": 1 },
            "completed_requests": {
              "$sum": {
                "$cond": [
                  { "$eq": [ "$basic_info.state.name", "Close" ] },
                  1,
                  0
                ]
              }
            },
            "in_progress_requests": {
              "$sum": {
                "$cond": [
                  { "$eq": [ "$basic_info.state.name", "In Progress" ] },
                  1,
                  0
                ]
              }
            },
            "avg_completion_time_hours": {
              "$avg": {
                "$cond": [
                  { "$ne": [ "$actual_closure_date", null ] },
                  { "$divide": [ { "$subtract": [ "$actual_closure_date", "$creation_time" ] }, 3600000 ] },
                  null
                ]
              }
            },
            "recent_usage": { "$max": "$creation_time" }
          }
        }
      },
      {
        "comment": "STEP 3: Calculate completion percentage.",
        "stage": {
          "$project": {
            "_id": 1,
            "total_requests": 1,
            "completed_requests": 1,
            "in_progress_requests": 1,
            "avg_completion_time_hours": 1,
            "recent_usage": 1,
            "completion_percentage": {
              "$multiply": [
                { "$divide": [ "$completed_requests", "$total_requests" ] },
                100
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by total requests descending.",
        "stage": {
          "$sort": {
            "total_requests": -1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find overdue tickets with aging analysis",
    "description": "Identify tickets that are overdue and analyze their aging patterns.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match open tickets older than 7 days.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.state.name": { "$ne": "Close" },
            "creation_time": {
              "$lte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "day",
                  "amount": 7
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Calculate aging and categorize tickets.",
        "stage": {
          "$project": {
            "_id": 0,
            "display_id": 1,
            "basic_info.summary": 1,
            "basic_info.priority.name": 1,
            "basic_info.state.name": 1,
            "creation_time": 1,
            "requester.full_name": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "current_assignment_info.group_name": 1,
            "age_days": {
              "$divide": [
                { "$subtract": [ "{{dt_now}}", "$creation_time" ] },
                86400000
              ]
            },
            "age_category": {
              "$switch": {
                "branches": [
                  {
                    "case": { "$gte": [ { "$divide": [ { "$subtract": [ "{{dt_now}}", "$creation_time" ] }, 86400000 ] }, 30 ] },
                    "then": "Critical (30+ days)"
                  },
                  {
                    "case": { "$gte": [ { "$divide": [ { "$subtract": [ "{{dt_now}}", "$creation_time" ] }, 86400000 ] }, 14 ] },
                    "then": "High (14-29 days)"
                  },
                  {
                    "case": { "$gte": [ { "$divide": [ { "$subtract": [ "{{dt_now}}", "$creation_time" ] }, 86400000 ] }, 7 ] },
                    "then": "Medium (7-13 days)"
                  }
                ],
                "default": "Low (0-6 days)"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Union with overdue requests.",
        "stage": {
          "$unionWith": {
            "coll": "request_process",
            "pipeline": [
              {
                "$match": {
                  "is_deleted": false,
                  "basic_info.state.name": { "$ne": "Close" },
                  "creation_time": {
                    "$lte": {
                      "$dateSubtract": {
                        "startDate": "{{dt_now}}",
                        "unit": "day",
                        "amount": 7
                      }
                    }
                  }
                }
              },
              {
                "$project": {
                  "_id": 0,
                  "display_id": 1,
                  "basic_info.summary": 1,
                  "basic_info.priority.name": 1,
                  "basic_info.state.name": 1,
                  "creation_time": 1,
                  "requester.full_name": 1,
                  "current_assignment_info.assignee_profile.full_name": 1,
                  "current_assignment_info.group_name": 1,
                  "age_days": {
                    "$divide": [
                      { "$subtract": [ "{{dt_now}}", "$creation_time" ] },
                      86400000
                    ]
                  },
                  "age_category": {
                    "$switch": {
                      "branches": [
                        {
                          "case": { "$gte": [ { "$divide": [ { "$subtract": [ "{{dt_now}}", "$creation_time" ] }, 86400000 ] }, 30 ] },
                          "then": "Critical (30+ days)"
                        },
                        {
                          "case": { "$gte": [ { "$divide": [ { "$subtract": [ "{{dt_now}}", "$creation_time" ] }, 86400000 ] }, 14 ] },
                          "then": "High (14-29 days)"
                        },
                        {
                          "case": { "$gte": [ { "$divide": [ { "$subtract": [ "{{dt_now}}", "$creation_time" ] }, 86400000 ] }, 7 ] },
                          "then": "Medium (7-13 days)"
                        }
                      ],
                      "default": "Low (0-6 days)"
                    }
                  }
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 4: Sort by age descending.",
        "stage": {
          "$sort": {
            "age_days": -1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get ticket volume trends by source for the last 6 months",
    "description": "Analyze ticket creation trends grouped by source over the past 6 months.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match tickets from the last 6 months.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": {
                "$dateSubtract": {
                  "startDate": "{{dt_now}}",
                  "unit": "month",
                  "amount": 6
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by source and month.",
        "stage": {
          "$group": {
            "_id": {
              "source": "$basic_info.incident_source.name",
              "year": { "$year": "$creation_time" },
              "month": { "$month": "$creation_time" }
            },
            "ticket_count": { "$sum": 1 },
            "avg_priority_score": {
              "$avg": {
                "$switch": {
                  "branches": [
                    { "case": { "$eq": [ "$basic_info.priority.name", "Critical" ] }, "then": 4 },
                    { "case": { "$eq": [ "$basic_info.priority.name", "High" ] }, "then": 3 },
                    { "case": { "$eq": [ "$basic_info.priority.name", "Medium" ] }, "then": 2 },
                    { "case": { "$eq": [ "$basic_info.priority.name", "Low" ] }, "then": 1 }
                  ],
                  "default": 0
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Union with request data.",
        "stage": {
          "$unionWith": {
            "coll": "request_process",
            "pipeline": [
              {
                "$match": {
                  "is_deleted": false,
                  "creation_time": {
                    "$gte": {
                      "$dateSubtract": {
                        "startDate": "{{dt_now}}",
                        "unit": "month",
                        "amount": 6
                      }
                    }
                  }
                }
              },
              {
                "$group": {
                  "_id": {
                    "source": "$basic_info.request_source.name",
                    "year": { "$year": "$creation_time" },
                    "month": { "$month": "$creation_time" }
                  },
                  "ticket_count": { "$sum": 1 },
                  "avg_priority_score": {
                    "$avg": {
                      "$switch": {
                        "branches": [
                          { "case": { "$eq": [ "$basic_info.priority.name", "Critical" ] }, "then": 4 },
                          { "case": { "$eq": [ "$basic_info.priority.name", "High" ] }, "then": 3 },
                          { "case": { "$eq": [ "$basic_info.priority.name", "Medium" ] }, "then": 2 },
                          { "case": { "$eq": [ "$basic_info.priority.name", "Low" ] }, "then": 1 }
                        ],
                        "default": 0
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 4: Group again to merge data from both collections.",
        "stage": {
          "$group": {
            "_id": "$_id",
            "total_tickets": { "$sum": "$ticket_count" },
            "avg_priority_score": { "$avg": "$avg_priority_score" }
          }
        }
      },
      {
        "comment": "STEP 5: Sort by year, month, and source.",
        "stage": {
          "$sort": {
            "_id.year": 1,
            "_id.month": 1,
            "_id.source": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get requester activity summary for email {{email}}",
    "description": "Analyze ticket creation patterns and activity for a specific requester email.",
    "collection": "incident",
    "pipeline": [
      {
        "comment": "STEP 1: Match tickets from the specific requester email.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "requester.email": "{{email}}"
          }
        }
      },
      {
        "comment": "STEP 2: Group by requester and calculate activity metrics.",
        "stage": {
          "$group": {
            "_id": "$requester.email",
            "requester_name": { "$first": "$requester.full_name" },
            "total_tickets": { "$sum": 1 },
            "open_tickets": {
              "$sum": {
                "$cond": [
                  { "$ne": [ "$basic_info.state.name", "Close" ] },
                  1,
                  0
                ]
              }
            },
            "closed_tickets": {
              "$sum": {
                "$cond": [
                  { "$eq": [ "$basic_info.state.name", "Close" ] },
                  1,
                  0
                ]
              }
            },
            "high_priority_tickets": {
              "$sum": {
                "$cond": [
                  { "$eq": [ "$basic_info.priority.name", "High" ] },
                  1,
                  0
                ]
              }
            },
            "first_ticket_date": { "$min": "$creation_time" },
            "last_ticket_date": { "$max": "$creation_time" },
            "services_used": { "$addToSet": "$basic_info.impact_service_name" }
          }
        }
      },
      {
        "comment": "STEP 3: Union with request activity data.",
        "stage": {
          "$unionWith": {
            "coll": "request_process",
            "pipeline": [
              {
                "$match": {
                  "is_deleted": false,
                  "requester.email": "{{email}}"
                }
              },
              {
                "$group": {
                  "_id": "$requester.email",
                  "requester_name": { "$first": "$requester.full_name" },
                  "total_tickets": { "$sum": 1 },
                  "open_tickets": {
                    "$sum": {
                      "$cond": [
                        { "$ne": [ "$basic_info.state.name", "Close" ] },
                        1,
                        0
                      ]
                    }
                  },
                  "closed_tickets": {
                    "$sum": {
                      "$cond": [
                        { "$eq": [ "$basic_info.state.name", "Close" ] },
                        1,
                        0
                      ]
                    }
                  },
                  "high_priority_tickets": {
                    "$sum": {
                      "$cond": [
                        { "$eq": [ "$basic_info.priority.name", "High" ] },
                        1,
                        0
                      ]
                    }
                  },
                  "first_ticket_date": { "$min": "$creation_time" },
                  "last_ticket_date": { "$max": "$creation_time" },
                  "services_used": { "$addToSet": "$basic_info.impact_service_name" }
                }
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 4: Group again to merge data from both collections.",
        "stage": {
          "$group": {
            "_id": "$_id",
            "requester_name": { "$first": "$requester_name" },
            "total_tickets": { "$sum": "$total_tickets" },
            "open_tickets": { "$sum": "$open_tickets" },
            "closed_tickets": { "$sum": "$closed_tickets" },
            "high_priority_tickets": { "$sum": "$high_priority_tickets" },
            "first_ticket_date": { "$min": "$first_ticket_date" },
            "last_ticket_date": { "$max": "$last_ticket_date" },
            "services_used": { "$first": "$services_used" }
          }
        }
      },
      {
        "comment": "STEP 5: Calculate activity duration and format output.",
        "stage": {
          "$project": {
            "_id": 0,
            "requester_email": "$_id",
            "requester_name": 1,
            "total_tickets": 1,
            "open_tickets": 1,
            "closed_tickets": 1,
            "high_priority_tickets": 1,
            "first_ticket_date": 1,
            "last_ticket_date": 1,
            "services_used": 1,
            "activity_duration_days": {
              "$divide": [
                { "$subtract": [ "$last_ticket_date", "$first_ticket_date" ] },
                86400000
              ]
            }
          }
        }
      }
    ]
  }
]
```