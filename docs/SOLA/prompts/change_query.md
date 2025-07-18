# Generate Queries from Metadata
You are a senior MongoDB architect and prompt‑engineer.

## Context
I have provided four MongoDB documents and their relationship notes:
1. `change` – definition of each change

Those JSON docs are **exactly as pasted** below this message.  
Read them carefully; do not guess any additional fields.

## Task
Generate a **comprehensive list of useful analytics questions** that an Operation and Technician engineer, Change Manager, or data analyst might ask about incident and request.  
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
    "nl_query": "What is the total number of changes created in the last 30 days",
    "description": "Counts all changes created within the last 30 days.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes for the organization created in last 30 days.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "day", "amount": 60 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Count the documents.",
        "stage": {
          "$count": "total_changes"
        }
      }
    ]
  },
  {
    "nl_query": "What is the average lead time (in hours) for changes closed in the last 90 days",
    "description": "Computes average of the existing lead_time field for closed changes in the last 90 days.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match closed changes with closure time in last 90 days.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "change_closure_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "day", "amount": 90 } }
            },
            "lead_time": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Compute average lead time.",
        "stage": {
          "$group": {
            "_id": null,
            "avg_lead_time_hours": { "$avg": "$lead_time" }
          }
        }
      }
    ]
  },
  {
    "nl_query": "How many changes are linked to a specific impacted service {{service}}?",
    "description": "Counts changes where basic_info.impact_service equals the given service name.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes for given impacted service.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.impact_servicename": "{{service}}"
          }
        }
      },
      {
        "comment": "STEP 2: Count the matched changes.",
        "stage": {
          "$count": "changes_with_service"
        }
      }
    ]
  },
  {
    "nl_query": "What is the count of high-risk changes by month for the last 6 months?",
    "description": "Groups changes marked as high-risk by month.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match high-risk changes in last 6 months.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.risk.name": "High",
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "month", "amount": 6 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by month and count.",
        "stage": {
          "$group": {
            "_id": {
              "year": { "$year": "$creation_time" },
              "month": { "$month": "$creation_time" }
            },
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by date ascending.",
        "stage": {
          "$sort": { "_id.year": 1, "_id.month": 1 }
        }
      }
    ]
  },
  {
    "nl_query": "What percentage of changes closed before due date in the last quarter",
    "description": "Computes SLA compliance for on-time closure.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match closed changes in the last quarter.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "change_closure_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "month", "amount": 3 } }
            },
            "due_date": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Add field whether closed on time.",
        "stage": {
          "$addFields": {
            "on_time": { "$cond": [{ "$lte": ["$change_closure_time", "$due_date"] }, 1, 0] }
          }
        }
      },
      {
        "comment": "STEP 3: Group and calculate percentage.",
        "stage": {
          "$group": {
            "_id": null,
            "total": { "$sum": 1 },
            "on_time_count": { "$sum": "$on_time" }
          }
        }
      },
      {
        "comment": "STEP 4: Project compliance percentage.",
        "stage": {
          "$project": {
            "_id": 0,
            "compliance_percentage": {
              "$multiply": [{ "$divide": ["$on_time_count", "$total"] }, 100]
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Top 5 services with the most linked changes in the past year.",
    "description": "Finds top impacted services from change data.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes for last 12 months.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "year", "amount": 1 } }
            },
            "basic_info.impact_service": { "$exists": true, "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Group by impact service and count.",
        "stage": {
          "$group": {
            "_id": "$basic_info.impact_service",
            "change_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count desc and limit 5.",
        "stage": {
          "$sort": { "change_count": -1 }
        }
      },
      {
        "comment": "STEP 4: Join with impact_service collection to get names.",
        "stage": {
          "$lookup": {
            "from": "impact_service",
            "localField": "_id",
            "foreignField": "service_id",
            "as": "service_info"
          }
        }
      },
      {
        "comment": "STEP 5: Project final output with service name.",
        "stage": {
          "$project": {
            "service_id": "$_id",
            "change_count": 1,
            "service_name": { "$arrayElemAt": ["$service_info.name", 0] }
          }
        }
      }
    ]
  },
  {
    "nl_query": "List all risks linked to high-priority changes in the past 3 months.",
    "description": "Fetches changes with high priority and risk in the last 3 months.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes with high priority and risk in last 3 months.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.priority": "High",
            "basic_info.risk": { "$exists": true, "$ne": null },
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "month", "amount": 3 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Project risk and change details.",
        "stage": {
          "$project": {
            "_id": 0,
            "change_id": 1,
            "display_id": 1,
            "risk": "$basic_info.risk",
            "priority": "$basic_info.priority"
          }
        }
      }
    ]
  },
   {
    "nl_query": "What is the distribution of changes by change type in the last 6 months?",
    "description": "Groups changes by change_type to show the distribution of Emergency, Normal, Standard, and other types.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes created in the last 6 months.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "month", "amount": 6 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by change type and count.",
        "stage": {
          "$group": {
            "_id": "$basic_info.change_type.name",
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending.",
        "stage": {
          "$sort": { "count": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Which change managers have the most assigned changes currently in progress?",
    "description": "Finds change managers with the highest number of active in-progress changes.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes currently in progress state.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.state.name": "In Progress"
          }
        }
      },
      {
        "comment": "STEP 2: Group by change manager and count.",
        "stage": {
          "$group": {
            "_id": {
              "manager_id": "$change_manager.profile_id",
              "manager_name": "$change_manager.full_name"
            },
            "active_changes": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by active changes descending.",
        "stage": {
          "$sort": { "active_changes": -1 }
        }
      },
      {
        "comment": "STEP 4: Project final output.",
        "stage": {
          "$project": {
            "_id": 0,
            "manager_id": "$_id.manager_id",
            "manager_name": "$_id.manager_name",
            "active_changes": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "What is the average time between plan start date and actual start date for implemented changes?",
    "description": "Calculates the variance between planned and actual start times for changes that have been implemented.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes with both planned and actual start dates.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.plan_start_date": { "$ne": null },
            "actual_start_date": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Calculate time difference in hours.",
        "stage": {
          "$addFields": {
            "variance_hours": {
              "$divide": [
                { "$subtract": ["$actual_start_date", "$basic_info.plan_start_date"] },
                3600000
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Group and calculate average variance.",
        "stage": {
          "$group": {
            "_id": null,
            "avg_variance_hours": { "$avg": "$variance_hours" },
            "total_changes": { "$sum": 1 }
          }
        }
      }
    ]
  },
  {
    "nl_query": "What are the top 5 locations with the most emergency changes in the last year?",
    "description": "Identifies locations with highest emergency change volume for capacity planning.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match emergency changes in the last year.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.change_type.name": "Emergency",
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "year", "amount": 1 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by location and count.",
        "stage": {
          "$group": {
            "_id": {
              "location_id": "$location.location_id",
              "location_name": "$location.location_name",
              "city": "$location.city",
              "state": "$location.state"
            },
            "emergency_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by count descending and limit to top 5.",
        "stage": {
          "$sort": { "emergency_count": -1 }
        }
      },
      {
        "comment": "STEP 4: Limit to top 5 locations.",
        "stage": {
          "$limit": 5
        }
      },
      {
        "comment": "STEP 5: Project final output.",
        "stage": {
          "$project": {
            "_id": 0,
            "location_id": "$_id.location_id",
            "location_name": "$_id.location_name",
            "city": "$_id.city",
            "state": "$_id.state",
            "emergency_count": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "How many changes are currently assigned to each support group?",
    "description": "Shows current workload distribution across different support groups.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match active changes not in closed state.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.state.name": { "$nin": ["Closed", "Cancelled"] }
          }
        }
      },
      {
        "comment": "STEP 2: Group by support group and count.",
        "stage": {
          "$group": {
            "_id": {
              "group_id": "$current_assignment_info.group",
              "group_name": "$current_assignment_info.group_name",
              "group_type": "$current_assignment_info.group_type"
            },
            "active_changes": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by active changes descending.",
        "stage": {
          "$sort": { "active_changes": -1 }
        }
      },
      {
        "comment": "STEP 4: Project final output.",
        "stage": {
          "$project": {
            "_id": 0,
            "group_id": "$_id.group_id",
            "group_name": "$_id.group_name",
            "group_type": "$_id.group_type",
            "active_changes": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "What is the monthly trend of changes by priority level for the last 12 months?",
    "description": "Shows monthly trends of changes grouped by priority (High, Medium, Low) for trend analysis.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes in the last 12 months.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "year", "amount": 1 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by month and priority.",
        "stage": {
          "$group": {
            "_id": {
              "year": { "$year": "$creation_time" },
              "month": { "$month": "$creation_time" },
              "priority": "$basic_info.priority.name"
            },
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by date ascending.",
        "stage": {
          "$sort": { "_id.year": 1, "_id.month": 1, "_id.priority": 1 }
        }
      }
    ]
  },
  {
    "nl_query": "What is the average response time for changes by change source (Web, Email, etc.)?",
    "description": "Computes average response time grouped by change source to identify efficiency by channel.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes with response time and source.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "change_response_time": { "$ne": null },
            "creation_time": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Calculate response time in hours.",
        "stage": {
          "$addFields": {
            "response_time_hours": {
              "$divide": [
                { "$subtract": ["$change_response_time", "$creation_time"] },
                3600000
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Group by change source and calculate average.",
        "stage": {
          "$group": {
            "_id": "$basic_info.change_source.name",
            "avg_response_time_hours": { "$avg": "$response_time_hours" },
            "total_changes": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by average response time.",
        "stage": {
          "$sort": { "avg_response_time_hours": 1 }
        }
      }
    ]
  },
  {
    "nl_query": "What is the average planning duration (plan_end_date - plan_start_date) by change type?",
    "description": "Calculates average planned duration for changes grouped by change type for planning insights.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes with both plan start and end dates.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.plan_start_date": { "$ne": null },
            "basic_info.plan_end_date": { "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Calculate planning duration in hours.",
        "stage": {
          "$addFields": {
            "planning_duration_hours": {
              "$divide": [
                { "$subtract": ["$basic_info.plan_end_date", "$basic_info.plan_start_date"] },
                3600000
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Group by change type and calculate average.",
        "stage": {
          "$group": {
            "_id": "$basic_info.change_type.name",
            "avg_planning_duration_hours": { "$avg": "$planning_duration_hours" },
            "total_changes": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by average duration descending.",
        "stage": {
          "$sort": { "avg_planning_duration_hours": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Find changes created by each requester in the last quarter and their current status.",
    "description": "Shows change creation patterns by requester with status distribution for workload analysis.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes from last quarter.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "month", "amount": 3 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by requester and status.",
        "stage": {
          "$group": {
            "_id": {
              "requester_id": "$requester.requester_id",
              "requester_name": "$requester.full_name",
              "requester_email": "$requester.email",
              "status": "$basic_info.state.name"
            },
            "count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Group by requester to get total and status breakdown.",
        "stage": {
          "$group": {
            "_id": {
              "requester_id": "$_id.requester_id",
              "requester_name": "$_id.requester_name",
              "requester_email": "$_id.requester_email"
            },
            "total_changes": { "$sum": "$count" },
            "status_breakdown": {
              "$push": {
                "status": "$_id.status",
                "count": "$count"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by total changes descending.",
        "stage": {
          "$sort": { "total_changes": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "What are the most common planning descriptions used in changes?",
    "description": "Analyzes planning field content to identify common patterns in change planning descriptions.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes with planning descriptions.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "basic_info.planning": { "$exists": true, "$ne": null, "$ne": "" }
          }
        }
      },
      {
        "comment": "STEP 2: Extract text from HTML planning field.",
        "stage": {
          "$addFields": {
            "planning_text": {
              "$replaceAll": {
                "input": {
                  "$replaceAll": {
                    "input": "$basic_info.planning",
                    "find": "&lt;",
                    "replacement": "<"
                  }
                },
                "find": "&gt;",
                "replacement": ">"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Group by planning text and count.",
        "stage": {
          "$group": {
            "_id": "$planning_text",
            "usage_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by usage count descending.",
        "stage": {
          "$sort": { "usage_count": -1 }
        }
      },
      {
        "comment": "STEP 5: Limit to top 10 most common.",
        "stage": {
          "$limit": 10
        }
      }
    ]
  },
  {
    "nl_query": "Find changes that have followers (watchers) and count followers per change.",
    "description": "Identifies changes with watchers to understand stakeholder engagement patterns.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes with watchers.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "current_watcher": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Add watcher count field.",
        "stage": {
          "$addFields": {
            "watcher_count": { "$size": "$current_watcher" }
          }
        }
      },
      {
        "comment": "STEP 3: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "change_id": 1,
            "display_id": 1,
            "basic_info.summary": 1,
            "basic_info.state.name": 1,
            "watcher_count": 1,
            "watchers": "$current_watcher"
          }
        }
      },
      {
        "comment": "STEP 4: Sort by watcher count descending.",
        "stage": {
          "$sort": { "watcher_count": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "What is the distribution of changes by catalogue and category in the last year?",
    "description": "Shows change distribution across service catalogues and categories for service analysis.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes from last year.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "year", "amount": 1 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by catalogue and category.",
        "stage": {
          "$group": {
            "_id": {
              "catalogue_name": "$basic_info.catalogue_name",
              "category_name": "$basic_info.category_name"
            },
            "change_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by change count descending.",
        "stage": {
          "$sort": { "change_count": -1 }
        }
      },
      {
        "comment": "STEP 4: Project final output.",
        "stage": {
          "$project": {
            "_id": 0,
            "catalogue_name": "$_id.catalogue_name",
            "category_name": "$_id.category_name",
            "change_count": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find changes that are overdue (past due_date) and still not closed.",
    "description": "Identifies overdue changes for prioritization and escalation purposes.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match changes that are overdue and not closed.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "due_date": { "$ne": null, "$lt": "{{dt_now}}" },
            "basic_info.state.name": { "$nin": ["Closed", "Cancelled"] }
          }
        }
      },
      {
        "comment": "STEP 2: Calculate overdue hours.",
        "stage": {
          "$addFields": {
            "overdue_hours": {
              "$divide": [
                { "$subtract": ["{{dt_now}}", "$due_date"] },
                3600000
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "change_id": 1,
            "display_id": 1,
            "basic_info.summary": 1,
            "basic_info.state.name": 1,
            "basic_info.priority.name": 1,
            "current_assignment_info.group_name": 1,
            "current_assignment_info.assignee_profile.full_name": 1,
            "due_date": 1,
            "overdue_hours": 1
          }
        }
      },
      {
        "comment": "STEP 4: Sort by overdue hours descending.",
        "stage": {
          "$sort": { "overdue_hours": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "What is the percentage of changes that have rollback plans documented?",
    "description": "Calculates the percentage of changes with documented rollback plans for compliance tracking.",
    "collection": "change",
    "pipeline": [
      {
        "comment": "STEP 1: Match all changes in the specified period.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "{{dt_now}}", "unit": "month", "amount": 6 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Add field indicating if rollback is documented.",
        "stage": {
          "$addFields": {
            "has_rollback_plan": {
              "$cond": [
                { "$and": [
                  { "$ne": ["$basic_info.rollback", null] },
                  { "$ne": ["$basic_info.rollback", ""] },
                  { "$ne": ["$basic_info.rollback", "&lt;p&gt;N/A&lt;/p&gt;"] }
                ]},
                1,
                0
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Group and calculate percentage.",
        "stage": {
          "$group": {
            "_id": null,
            "total_changes": { "$sum": 1 },
            "with_rollback_plan": { "$sum": "$has_rollback_plan" }
          }
        }
      },
      {
        "comment": "STEP 4: Calculate percentage.",
        "stage": {
          "$project": {
            "_id": 0,
            "total_changes": 1,
            "with_rollback_plan": 1,
            "rollback_compliance_percentage": {
              "$multiply": [
                { "$divide": ["$with_rollback_plan", "$total_changes"] },
                100
              ]
            }
          }
        }
      }
    ]
  }
]