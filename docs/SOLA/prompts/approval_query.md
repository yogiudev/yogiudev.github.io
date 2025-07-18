# Generate Queries from Metadata

You are a senior MongoDB architect and prompt‑engineer.

## Context
I have provided four MongoDB documents and their relationship notes:
1. `approval_requests` – captures the approval flow for entities like tickets, changes, etc.
2. `teams` – definition of approval teams, including members and structure.
3. `staffs` – all users (approvers and requesters) and their associated details.
4. `sequence_staffs` – represents individual approvers within each team, with sequence and delegation settings.

These documents are **exactly as pasted** below this message.  
Read them carefully; do not guess any additional fields.

## Task
Generate a **comprehensive list of useful analytics questions** that an ops engineer, team lead, or compliance auditor might ask about this approval system.  
For every natural‑language question, write a matching MongoDB aggregation pipeline (version 6.x syntax).

### Requirements
* Cover at least **20** distinct questions; aim for variety (e.g., approval time, bottlenecks, delegation impact, pending count per user, etc.)
* Use `$lookup` joins where cross‑collection data is required; minimize client‑side post‑processing.
* Use **stage comments** (`/* … */`) to explain logic inside each pipeline.
* Filters (e.g., `team_id`, `requested_by`, `status`, `created_at`) must be dynamic.
* Use placeholder variables like `{{team_id}}`, `{{requested_by}}`, `{{start_date}}`, `{{end_date}}`, `{{status}}`.

### Output format
Return a **single JSON array**.  
Each element must have this shape:

```jsonc
[
  {
    "nl_query": "Which users had the most pending approvals between {{start_date}} and {{end_date}}?",
    "description": "Count how many approval_requests are currently pending for each staff member during the given period.",
    "collection": "approval_requests",
    "pipeline": [
      {
        "comment": "STEP 1: Filter approval requests by date and pending status.",
        "stage": {
          "$match": {
            "is_deleted": false,
            "created_at": {
              "$gte": "{{start_date}}",
              "$lte": "{{end_date}}"
            },
            "status": "Pending"
          }
        }
      },
      {
        "comment": "STEP 2: Join with sequence_staffs to get approver details.",
        "stage": {
          "$lookup": {
            "from": "sequence_staffs",
            "localField": "_id",
            "foreignField": "approval_request_id",
            "as": "approver_sequence"
          }
        }
      },
      {
        "comment": "STEP 3: Unwind the sequence array.",
        "stage": {
          "$unwind": "$approver_sequence"
        }
      },
      {
        "comment": "STEP 4: Group by approver and count.",
        "stage": {
          "$group": {
            "_id": "$approver_sequence.staff_id",
            "pending_count": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 5: Lookup staff name and email.",
        "stage": {
          "$lookup": {
            "from": "staffs",
            "localField": "_id",
            "foreignField": "_id",
            "as": "staff"
          }
        }
      },
      {
        "comment": "STEP 6: Flatten staff details.",
        "stage": {
          "$unwind": "$staff"
        }
      },
      {
        "comment": "STEP 7: Project final fields.",
        "stage": {
          "$project": {
            "staff_id": "$_id",
            "full_name": "$staff.full_name",
            "email": "$staff.email",
            "pending_count": 1,
            "_id": 0
          }
        }
      },
      {
        "comment": "STEP 8: Sort descending by pending_count.",
        "stage": {
          "$sort": { "pending_count": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "What are all pending approval requests in organization {{organization_id}}?",
    "description": "Retrieve all approval requests that are currently pending approval in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match pending approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "overall_approval_state": 0,
            "is_active": true,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sequence": 1,
            "submitted_by": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show all incident approval requests in organization {{organization_id}}?",
    "description": "Find all approval requests related to incidents (module_id = 10) in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match incident approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "module_id": 10,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "is_approval_completed": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
    {
    "nl_query": "Show all request process approval requests in organization {{organization_id}}?",
    "description": "Find all approval requests related to request processes (module_id = 42) in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match request process approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "module_id": 42,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "is_approval_completed": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show all problem approval requests in organization {{organization_id}}?",
    "description": "Find all problem approval requests related to problem processes (module_id = 45) in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match problem approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "module_id": 45,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "is_approval_completed": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show all change approval requests in organization {{organization_id}}?",
    "description": "Find all change approval requests (module_id = 47) in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match change approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "module_id": 47,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "is_approval_completed": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Which approval requests in organization {{organization_id}} have been rejected?",
    "description": "List all approval requests that have been rejected (overall_approval_state = 2) in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match rejected approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "overall_approval_state": 2,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields with rejection details.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "submitted_by": 1,
            "creation_date": 1,
            "last_updated": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval statistics by module for organization {{organization_id}}?",
    "description": "Aggregate approval request statistics grouped by module_id for the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Group by module and calculate statistics.",
        "stage": {
          "$group": {
            "_id": "$module_id",
            "total_requests": { "$sum": 1 },
            "pending": { "$sum": { "$cond": [{ "$eq": ["$overall_approval_state", 0] }, 1, 0] } },
            "approved": { "$sum": { "$cond": [{ "$eq": ["$overall_approval_state", 1] }, 1, 0] } },
            "rejected": { "$sum": { "$cond": [{ "$eq": ["$overall_approval_state", 2] }, 1, 0] } }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find overdue approval requests in organization {{organization_id}}?",
    "description": "Retrieve pending approval requests older than 7 days in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match overdue pending approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "overall_approval_state": 0,
            "is_active": true,
            "is_deleted": false,
            "creation_date": { "$lt": { "$dateSubtract": { "startDate": "$$NOW", "unit": "day", "amount": 7 } } }
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields with age calculation.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sequence": 1,
            "creation_date": 1,
            "days_pending": { "$divide": [{ "$subtract": ["$$NOW", "$creation_date"] }, 86400000] }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approvals assigned to specific team in organization {{organization_id}}?",
    "description": "Find all approval requests assigned to a specific team within the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization and team.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_team.team_id": "{{team_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get completed approval requests in organization {{organization_id}}?",
    "description": "Retrieve all approval requests that have been completed in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match completed approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_approval_completed": true,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "submitted_by": 1,
            "creation_date": 1,
            "last_updated": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests requiring response in organization {{organization_id}}?",
    "description": "List all approval requests where response_required is true in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals requiring response by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "response_required": true,
            "overall_approval_state": 0,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sequence": 1,
            "response_required": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests by specific submitter in organization {{organization_id}}?",
    "description": "Find all approval requests submitted by a specific user in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization and submitter.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "submitted_by": "{{profile_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests at specific sequence in organization {{organization_id}}?",
    "description": "Find all approval requests currently at a specific approval sequence in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization and sequence.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sequence": "{{sequence_number}}",
            "overall_approval_state": 0,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sequence": 1,
            "approval_sent_to": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find manual approval requests in organization {{organization_id}}?",
    "description": "Retrieve all manual approval requests (approval_type = 1) in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match manual approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_type": 1,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_type": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with partial approval requirements in organization {{organization_id}}?",
    "description": "Find approval requests where partial approvals are allowed (approval_percent_required > 0 and < 100).",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match partial approval requests by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_percent_required": { "$gt": 0, "$lt": 100 },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_percent_required": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests by workflow block in organization {{organization_id}}?",
    "description": "Find all approval requests associated with a specific workflow block in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization and workflow block.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_block_flow_id": "{{workflow_block_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_block_flow_id": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with comments in organization {{organization_id}}?",
    "description": "Find approval requests where approvers have provided comments in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization with comments.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approval_comment": { "$exists": true, "$ne": "" },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find active approval requests in organization {{organization_id}}?",
    "description": "Retrieve all currently active approval requests in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match active approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_active": true,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "is_active": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests created today in organization {{organization_id}}?",
    "description": "Find all approval requests created today in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match today's approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "creation_date": { 
              "$gte": { "$dateFromString": { "dateString": { "$dateToString": { "format": "%Y-%m-%d", "date": "$$NOW" } } } },
              "$lt": { "$dateAdd": { "startDate": { "$dateFromString": { "dateString": { "$dateToString": { "format": "%Y-%m-%d", "date": "$$NOW" } } } }, "unit": "day", "amount": 1 } }
            },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "submitted_by": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval progress report for organization {{organization_id}}?",
    "description": "Generate a progress report showing approval completion percentages for the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match active approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_active": true,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Add progress calculation.",
        "stage": {
          "$addFields": {
            "total_sequences": { "$size": "$approval_sent_to" },
            "progress_percentage": {
              "$multiply": [
                { "$divide": ["$approval_sequence", { "$size": "$approval_sent_to" }] },
                100
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
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sequence": 1,
            "total_sequences": 1,
            "progress_percentage": 1,
            "overall_approval_state": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests by date range in organization {{organization_id}}?",
    "description": "Retrieve approval requests created within a specific date range in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization and date range.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "creation_date": {
              "$gte": "{{start_date}}",
              "$lte": "{{end_date}}"
            },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "submitted_by": 1,
            "creation_date": 1,
            "last_updated": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with multiple sequences in organization {{organization_id}}?",
    "description": "Find approval requests that have multiple approval sequences in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization with multiple sequences.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.1": { "$exists": true },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields with sequence count.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sequence": 1,
            "sequence_count": { "$size": "$approval_sent_to" },
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests with specific approval source in organization {{organization_id}}?",
    "description": "Find approval requests based on their approval source (manual, rule-based, API) in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals by organization and source.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approval_source": "{{approval_source}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  
  {
    "nl_query": "Find approval requests with automated approval type in organization {{organization_id}}?",
    "description": "Retrieve all approval requests that are automated (approval_type = 2) in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match automated approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_type": 2,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_type": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show problem approval requests in organization {{organization_id}}?",
    "description": "Find all approval requests related to problems (module_id = 45) in the specified organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match problem approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "module_id": 45,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "approval_sequence": 1,
            "submitted_by": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests with 100% approval requirement in organization {{organization_id}}?",
    "description": "Find approval requests where full approval is required (approval_percent_required = 100) in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match full approval requirement by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_percent_required": 100,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_percent_required": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests with API source in organization {{organization_id}}?",
    "description": "Retrieve approval requests where at least one approval came from API source in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals with API source by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approval_source": 3,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with on-behalf approvals in organization {{organization_id}}?",
    "description": "Find approval requests where approvals were made on behalf of someone else in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match on-behalf approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approved_on_behalf": { "$exists": true, "$ne": null },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests by approval data ID in organization {{organization_id}}?",
    "description": "Find approval requests containing a specific approval_data_id in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific approval data ID by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approval_data_id": "{{approval_data_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests with workflow information in organization {{organization_id}}?",
    "description": "Retrieve approval requests that have previous workflow status information in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals with workflow info by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "previous_status.workflow_id": { "$exists": true, "$ne": null },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "previous_status": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with specific approver email in organization {{organization_id}}?",
    "description": "Find approval requests where a specific email address is an approver in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific approver email by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approved_by.email": "{{approver_email}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests with specific state colors in organization {{organization_id}}?",
    "description": "Find approval requests with specific color coding in their previous status in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific color by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "previous_status.color": "{{color_code}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "previous_status": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests modified this week in organization {{organization_id}}?",
    "description": "Retrieve approval requests that have been updated within the current week in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals modified this week by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "last_updated": { 
              "$gte": { "$dateSubtract": { "startDate": "$$NOW", "unit": "day", "amount": 7 } }
            },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "overall_approval_state": 1,
            "last_updated": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with rule-based source in organization {{organization_id}}?",
    "description": "Find approval requests where at least one approval came from rule-based source in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match rule-based approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approval_source": 2,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests with specific GUID in organization {{organization_id}}?",
    "description": "Find approval requests containing a specific GUID in their previous status in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific GUID by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "previous_status.guid": "{{guid_value}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "previous_status": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests with late responses in organization {{organization_id}}?",
    "description": "Retrieve approval requests where approval_date is significantly later than approval_request_date in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals with late responses by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to": {
              "$elemMatch": {
                "approval_date": { "$exists": true },
                "approval_request_date": { "$exists": true },
                "$expr": {
                  "$gt": [
                    { "$subtract": ["$approval_date", "$approval_request_date"] },
                    86400000
                  ]
                }
              }
            },
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with specific workflow name in organization {{organization_id}}?",
    "description": "Find approval requests associated with a specific workflow name in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific workflow name by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "previous_status.workflow_name": "{{workflow_name}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "previous_status": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests with background color coding in organization {{organization_id}}?",
    "description": "Find approval requests with specific background color in their previous status in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific background color by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "previous_status.background_color": "{{background_color}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "previous_status": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests with specific state ID in organization {{organization_id}}?",
    "description": "Retrieve approval requests with a specific state_id in their previous status in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific state ID by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "previous_status.state_id": "{{state_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "previous_status": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with no response required in organization {{organization_id}}?",
    "description": "Find approval requests where response_required is false in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals with no response required by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "response_required": false,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "response_required": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Get approval requests with specific approver sequence in organization {{organization_id}}?",
    "description": "Find approval requests where approvers are at a specific sequence number in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match specific approver sequence by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "approval_sent_to.approved_by.sequence": "{{sequence_number}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find approval requests with inactive status in organization {{organization_id}}?",
    "description": "Retrieve approval requests that are currently inactive (is_active = false) in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match inactive approvals by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_active": false,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "is_active": 1,
            "overall_approval_state": 1,
            "creation_date": 1,
            "last_updated": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval requests with empty comments in organization {{organization_id}}?",
    "description": "Find approval requests where approvers have not provided any comments in the organization.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approvals with empty comments by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "$or": [
              { "approval_sent_to.approval_comment": { "$exists": false } },
              { "approval_sent_to.approval_comment": "" },
              { "approval_sent_to.approval_comment": null }
            ],
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "approval_request_id": 1,
            "module_id": 1,
            "ref_id": 1,
            "approval_sent_to": 1,
            "overall_approval_state": 1,
            "creation_date": 1
          }
        }
      }
    ]
  }
]
```