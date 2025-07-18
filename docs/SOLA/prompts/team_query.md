# Generate Queries from Team Metadata
You are a senior MongoDB architect and prompt-engineer.

## Context
I have provided a MongoDB document representing a **Team** configuration used in ITSM modules (incident, request, change).

Each document contains:
- Unique `team_id`, `organization`, and `module_id`
- `is_approval_sequence` flag defining sequential vs. parallel approval
- `level_staff` structure defining approval levels and users
- Possible empty fields like `staffs`, `sequence_staffs`, which may be used if `level_staff` is not used
- `business_hr_profile` indicating business hour logic for SLA-aware modules

### Module IDs:
- `10` = Incident
- `42` = Request
- `47` = Change

These documents control **who can approve**, in **what order**, and **how many must approve** based on percentage (not defined in the doc, assumed externally or by default logic).

## Task
Generate a **comprehensive list of analytics and operational questions** that an ITSM workflow admin, reporting analyst, or compliance officer might ask regarding team configurations.

For each natural-language question, write a matching MongoDB aggregation pipeline (MongoDB 6.x syntax) using the `team` collection.

### Requirements
- Cover at least **15 unique questions**, with diversity across:
  - Approval structure insight
  - Organizational/team distribution
  - Module coverage
  - Notification setup
  - Staff participation
  - Sequential vs. parallel approval analysis
  - Level distribution
  - Users per level
- Include **$project**, **$unwind**, and **$lookup** (if you assume other collections like users/profiles)
- Use **stage comments** (`/* ... */`) for each step
- Use **placeholders** like `{{organization_id}}`, `{{module_id}}` for dynamic filtering
- Return a **single JSON array**, each element like below:

```jsonc
[
  {
    "nl_query": "Which teams in organization {{organization_id}} have sequential approval enabled?",
    "description": "List teams within a given organization where is_approval_sequence is true.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization and sequential approval.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_approval_sequence": true,
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "is_approval_sequence": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show me the distribution of teams across different modules for organization {{organization_id}}",
    "description": "Group teams by module_id to understand module coverage and count teams per module.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization and exclude deleted teams.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Group by module_id and count teams.",
        "stage": {
          "$group": {
            "_id": "$module_id",
            "module_name": { "$first": "$module_name" },
            "team_count": { "$sum": 1 },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by module_id for consistent ordering.",
        "stage": {
          "$sort": { "_id": 1 }
        }
      }
    ]
  },
  {
    "nl_query": "Which teams have the most approval levels configured in their level_staff structure for organization {{organization_id}}?",
    "description": "Analyze level_staff array to find teams with the highest number of approval levels.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match active teams with level_staff configuration.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff to access expertise levels.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel to count individual levels.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Group by team and count levels.",
        "stage": {
          "$group": {
            "_id": "$team_id",
            "team_name": { "$first": "$name" },
            "module_id": { "$first": "$module_id" },
            "organization": { "$first": "$organization" },
            "level_count": { "$sum": 1 },
            "levels": {
              "$push": {
                "level": "$level_staff.expertGroupLevel.level",
                "level_id": "$level_staff.expertGroupLevel.level_id"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 5: Sort by level count in descending order.",
        "stage": {
          "$sort": { "level_count": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams where approval_percent is less than 100% indicating partial approval acceptance for organization {{organization_id}}",
    "description": "Identify teams with partial approval thresholds in their level_staff configuration.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match active teams with level_staff.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff array.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Match levels with approval_percent less than 100.",
        "stage": {
          "$match": {
            "level_staff.expertGroupLevel.approval_percent": { "$lt": 100 }
          }
        }
      },
      {
        "comment": "STEP 5: Project relevant fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "level": "$level_staff.expertGroupLevel.level",
            "approval_percent": "$level_staff.expertGroupLevel.approval_percent",
            "user_count": { "$size": "$level_staff.expertGroupLevel.users" }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams that require additional notifications (is_add_notify_required = true) for organization {{organization_id}}",
    "description": "List teams configured to trigger additional notifications beyond standard workflow.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with additional notification requirement.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "is_add_notify_required": true
          }
        }
      },
      {
        "comment": "STEP 2: Project team details and notification settings.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "organization": 1,
            "is_add_notify_required": 1,
            "response_required": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Which users appear in the most teams across organization {{organization_id}}?",
    "description": "Analyze user participation across multiple teams to identify highly involved users.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Create a unified user array from all possible sources.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "all_users": {
              "$concatArrays": [
                { "$ifNull": ["$owner", []] },
                { "$ifNull": ["$staffs", []] },
                {
                  "$reduce": {
                    "input": { "$ifNull": ["$sequence_staffs", []] },
                    "initialValue": [],
                    "in": { "$concatArrays": ["$$value", "$$this.users"] }
                  }
                },
                {
                  "$reduce": {
                    "input": { "$ifNull": ["$level_staff", []] },
                    "initialValue": [],
                    "in": {
                      "$concatArrays": [
                        "$$value",
                        {
                          "$reduce": {
                            "input": "$$this.expertGroupLevel",
                            "initialValue": [],
                            "in": { "$concatArrays": ["$$value", "$$this.users"] }
                          }
                        }
                      ]
                    }
                  }
                }
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Unwind the combined user array.",
        "stage": {
          "$unwind": "$all_users"
        }
      },
      {
        "comment": "STEP 4: Group by user to count team participation.",
        "stage": {
          "$group": {
            "_id": "$all_users.profile_id",
            "full_name": { "$first": "$all_users.full_name" },
            "email": { "$first": "$all_users.email" },
            "team_count": { "$sum": 1 },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "team_name": "$name"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 5: Sort by team participation count.",
        "stage": {
          "$sort": { "team_count": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams with individual staff selection type that have empty staffs array for organization {{organization_id}}",
    "description": "Identify potential configuration issues where individual selection is enabled but no staff assigned.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with individual selection but empty staffs.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "staff_selection_type": "individual",
            "$or": [
              { "staffs": { "$exists": false } },
              { "staffs": { "$eq": [] } }
            ]
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant configuration details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "staff_selection_type": 1,
            "has_level_staff": { "$ne": ["$level_staff", []] },
            "has_sequence_staff": { "$ne": ["$sequence_staffs", []] },
            "is_approval_sequence": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams configured with business hour profiles for module {{module_id}} in organization {{organization_id}}",
    "description": "List teams that have business hour profiles configured for SLA-aware processing.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by module and organization with business hour profile.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "module_id": "{{module_id}}",
            "is_deleted": false,
            "business_hr_profile": { "$exists": true, "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Project team and business hour details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "organization": 1,
            "business_hr_profile": 1,
            "response_required": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Which teams have rule-based assignment configuration enabled for organization {{organization_id}}?",
    "description": "Identify teams with automated assignment rules configured in their config object.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with rule assignment enabled.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "config.is_rule_assigned": true
          }
        }
      },
      {
        "comment": "STEP 2: Project team and rule configuration.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "organization": 1,
            "rule_config": {
              "rule_id": "$config.rule_id",
              "rule_type": "$config.rule_type",
              "is_rule_assigned": "$config.is_rule_assigned"
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams where notify_mail or notify_phone arrays are configured at approval levels for organization {{organization_id}}",
    "description": "Identify teams with notification channels configured for specific approval levels.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with level_staff configuration.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff array.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Match levels with notification channels.",
        "stage": {
          "$match": {
            "$or": [
              { "level_staff.expertGroupLevel.notify_mail": { "$exists": true, "$ne": [] } },
              { "level_staff.expertGroupLevel.notify_phone": { "$exists": true, "$ne": [] } }
            ]
          }
        }
      },
      {
        "comment": "STEP 5: Project notification details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "level": "$level_staff.expertGroupLevel.level",
            "notify_mail": "$level_staff.expertGroupLevel.notify_mail",
            "notify_phone": "$level_staff.expertGroupLevel.notify_phone"
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show the average number of users per approval level across all teams for organization {{organization_id}}",
    "description": "Calculate statistics on user distribution across approval levels to understand team sizing.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match active teams with level_staff.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff array.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Calculate user count per level.",
        "stage": {
          "$project": {
            "team_id": 1,
            "level": "$level_staff.expertGroupLevel.level",
            "user_count": { "$size": "$level_staff.expertGroupLevel.users" }
          }
        }
      },
      {
        "comment": "STEP 5: Group by level and calculate statistics.",
        "stage": {
          "$group": {
            "_id": "$level",
            "avg_users": { "$avg": "$user_count" },
            "min_users": { "$min": "$user_count" },
            "max_users": { "$max": "$user_count" },
            "total_teams": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 6: Sort by level for consistent ordering.",
        "stage": {
          "$sort": { "_id": 1 }
        }
      }
    ]
  },
  {
    "nl_query": "Which teams are configured as preconfigured/system-generated teams for organization {{organization_id}}?",
    "description": "List teams that are marked as system-generated or preconfigured templates.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match preconfigured teams.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "is_preconfigure": true
          }
        }
      },
      {
        "comment": "STEP 2: Project team details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "organization": 1,
            "is_preconfigure": 1,
            "creation_time": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams with asset_tags configured for asset-based assignment in organization {{organization_id}}",
    "description": "Identify teams that have asset tags configured for asset ownership or scope-based routing.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with asset tags.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "asset_tags": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Project team and asset tag details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "organization": 1,
            "asset_tags": 1,
            "tags": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams where response_required is true but no approval sequence is configured for organization {{organization_id}}",
    "description": "Identify potential configuration issues where response is required but approval flow is unclear.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams requiring response.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "response_required": true
          }
        }
      },
      {
        "comment": "STEP 2: Check for missing approval configuration.",
        "stage": {
          "$match": {
            "$and": [
              {
                "$or": [
                  { "is_approval_sequence": false },
                  { "is_approval_sequence": { "$exists": false } }
                ]
              },
              {
                "$or": [
                  { "level_staff": { "$eq": [] } },
                  { "level_staff": { "$exists": false } }
                ]
              },
              {
                "$or": [
                  { "staffs": { "$eq": [] } },
                  { "staffs": { "$exists": false } }
                ]
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 3: Project problematic configuration.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "response_required": 1,
            "is_approval_sequence": 1,
            "has_level_staff": { "$ne": ["$level_staff", []] },
            "has_staffs": { "$ne": ["$staffs", []] }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams created in the last 30 days for organization {{organization_id}}",
    "description": "List recently created teams to track team configuration changes and additions.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization and recent creation.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "creation_time": {
              "$gte": { "$dateSubtract": { "startDate": "$$NOW", "unit": "day", "amount": 30 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Project team details with creation info.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "creation_time": 1,
            "staff_selection_type": 1,
            "is_approval_sequence": 1
          }
        }
      },
      {
        "comment": "STEP 3: Sort by creation time descending.",
        "stage": {
          "$sort": { "creation_time": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams grouped by group_type with counts for organization {{organization_id}}",
    "description": "Analyze team organization by group type (Department, Location, Function) to understand team structure.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match active teams with group_type.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "group_type": { "$exists": true, "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Group by group_type and count.",
        "stage": {
          "$group": {
            "_id": "$group_type",
            "team_count": { "$sum": 1 },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name",
                "module_id": "$module_id"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by team count descending.",
        "stage": {
          "$sort": { "team_count": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams with location-based configuration for specific location category {{location_category}} in organization {{organization_id}}",
    "description": "Identify teams configured for specific location categories for geographic routing.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with specific location category.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "location.location_category": "{{location_category}}"
          }
        }
      },
      {
        "comment": "STEP 2: Project team and location details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "organization": 1,
            "location": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Which teams have event threshold rules configured for organization {{organization_id}}?",
    "description": "List teams with event threshold rules for notifications or metrics monitoring.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with event threshold rules.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "events_thres_id": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Project team and threshold details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "organization": 1,
            "events_thres_id": 1,
            "threshold_count": { "$size": "$events_thres_id" }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Compare sequential vs parallel approval team configurations across modules for organization {{organization_id}}",
    "description": "Analyze the distribution of approval types across different ITSM modules.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match active teams.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Group by module and approval type.",
        "stage": {
          "$group": {
            "_id": {
              "module_id": "$module_id",
              "module_name": "$module_name",
              "is_approval_sequence": "$is_approval_sequence"
            },
            "team_count": { "$sum": 1 },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Reshape for better analysis.",
        "stage": {
          "$group": {
            "_id": {
              "module_id": "$_id.module_id",
              "module_name": "$_id.module_name"
            },
            "sequential_count": {
              "$sum": {
                "$cond": [{ "$eq": ["$_id.is_approval_sequence", true] }, "$team_count", 0]
              }
            },
            "parallel_count": {
              "$sum": {
                "$cond": [{ "$eq": ["$_id.is_approval_sequence", false] }, "$team_count", 0]
              }
            },
            "total_teams": { "$sum": "$team_count" }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by module_id.",
        "stage": {
          "$sort": { "_id.module_id": 1 }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams with the highest number of total users across all approval levels for organization {{organization_id}}",
    "description": "Calculate total user count across all approval structures to identify largest teams.",
    "collection": "teams",
    "pipeline": [
      {
        "comment": "STEP 1: Match active teams.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Calculate total user count from all sources.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "organization": 1,
            "total_users": {
              "$add": [
                { "$size": { "$ifNull": ["$owner", []] } },
                { "$size": { "$ifNull": ["$staffs", []] } },
                {
                  "$reduce": {
                    "input": { "$ifNull": ["$sequence_staffs", []] },
                    "initialValue": 0,
                    "in": { "$add": ["$$value", { "$size": "$$this.users" }] }
                  }
                },
                {
                  "$reduce": {
                    "input": { "$ifNull": ["$level_staff", []] },
                    "initialValue": 0,
                    "in": {
                      "$add": [
                        "$$value",
                        {
                          "$reduce": {
                            "input": "$$this.expertGroupLevel",
                            "initialValue": 0,
                            "in": { "$add": ["$$value", { "$size": "$$this.users" }] }
                          }
                        }
                      ]
                    }
                  }
                }
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by total users descending.",
        "stage": {
          "$sort": { "total_users": -1 }
        }
      },
      {
        "comment": "STEP 4: Limit to top 10 teams.",
        "stage": {
          "$limit": 10
        }
      }
    ]
  },
  {
    "nl_query": "Which teams in organization {{organization_id}} have approval_percent less than 100% across all their approval levels?",
    "description": "Identify teams with partial approval thresholds configured in their hierarchical approval structure.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff array.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Filter levels with partial approval percentages.",
        "stage": {
          "$match": {
            "level_staff.expertGroupLevel.approval_percent": { "$lt": 100 }
          }
        }
      },
      {
        "comment": "STEP 5: Group back by team to show all partial levels.",
        "stage": {
          "$group": {
            "_id": "$team_id",
            "team_name": { "$first": "$name" },
            "module_id": { "$first": "$module_id" },
            "partial_levels": {
              "$push": {
                "level": "$level_staff.expertGroupLevel.level",
                "approval_percent": "$level_staff.expertGroupLevel.approval_percent",
                "user_count": { "$size": "$level_staff.expertGroupLevel.users" }
              }
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams in organization {{organization_id}} that have been updated in the last 7 days",
    "description": "Track recent team configuration changes for audit and monitoring purposes.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization and recent updates.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "last_update_time": {
              "$gte": { "$dateSubtract": { "startDate": "$$NOW", "unit": "day", "amount": 7 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Project relevant fields with update info.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "last_update_time": 1,
            "creation_time": 1,
            "is_preconfigure": 1
          }
        }
      },
      {
        "comment": "STEP 3: Sort by last update time descending.",
        "stage": {
          "$sort": { "last_update_time": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with empty owner arrays",
    "description": "Identify teams that lack proper ownership assignment for governance compliance.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with empty or missing owners.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "$or": [
              { "owner": { "$exists": false } },
              { "owner": { "$eq": [] } }
            ]
          }
        }
      },
      {
        "comment": "STEP 2: Project team details for review.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "group_type": 1,
            "staff_selection_type": 1,
            "has_staffs": { "$ne": ["$staffs", []] },
            "has_level_staff": { "$ne": ["$level_staff", []] }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams in organization {{organization_id}} grouped by staff_selection_type with approval configuration analysis",
    "description": "Analyze how different staff selection types correlate with approval configurations.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Group by staff selection type.",
        "stage": {
          "$group": {
            "_id": "$staff_selection_type",
            "total_teams": { "$sum": 1 },
            "sequential_approval_count": {
              "$sum": { "$cond": [{ "$eq": ["$is_approval_sequence", true] }, 1, 0] }
            },
            "parallel_approval_count": {
              "$sum": { "$cond": [{ "$eq": ["$is_approval_sequence", false] }, 1, 0] }
            },
            "response_required_count": {
              "$sum": { "$cond": [{ "$eq": ["$response_required", true] }, 1, 0] }
            },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name",
                "module_id": "$module_id"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Calculate percentages.",
        "stage": {
          "$project": {
            "_id": 1,
            "total_teams": 1,
            "sequential_approval_count": 1,
            "parallel_approval_count": 1,
            "response_required_count": 1,
            "sequential_percentage": {
              "$multiply": [
                { "$divide": ["$sequential_approval_count", "$total_teams"] },
                100
              ]
            },
            "response_required_percentage": {
              "$multiply": [
                { "$divide": ["$response_required_count", "$total_teams"] },
                100
              ]
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with notify_mail or notify_phone configured but no users in those approval levels",
    "description": "Identify potential configuration issues where notifications are set but no approvers exist.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with level_staff.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff array.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Find levels with notifications but no users.",
        "stage": {
          "$match": {
            "$and": [
              {
                "$or": [
                  { "level_staff.expertGroupLevel.notify_mail": { "$exists": true, "$ne": [] } },
                  { "level_staff.expertGroupLevel.notify_phone": { "$exists": true, "$ne": [] } }
                ]
              },
              {
                "$or": [
                  { "level_staff.expertGroupLevel.users": { "$eq": [] } },
                  { "level_staff.expertGroupLevel.users": { "$exists": false } }
                ]
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 5: Project problematic configurations.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "level": "$level_staff.expertGroupLevel.level",
            "has_notify_mail": { "$ne": ["$level_staff.expertGroupLevel.notify_mail", []] },
            "has_notify_phone": { "$ne": ["$level_staff.expertGroupLevel.notify_phone", []] },
            "user_count": { "$size": { "$ifNull": ["$level_staff.expertGroupLevel.users", []] } }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval request completion rates by team for organization {{organization_id}} in the last 30 days",
    "description": "Analyze team performance in completing approval requests using both collections.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match recent approval requests by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "creation_date": {
              "$gte": { "$dateSubtract": { "startDate": "$$NOW", "unit": "day", "amount": 30 } }
            }
          }
        }
      },
      {
        "comment": "STEP 2: Group by team and calculate completion stats.",
        "stage": {
          "$group": {
            "_id": "$approval_team.team_id",
            "total_requests": { "$sum": 1 },
            "completed_requests": {
              "$sum": { "$cond": [{ "$eq": ["$is_approval_completed", true] }, 1, 0] }
            },
            "approved_requests": {
              "$sum": { "$cond": [{ "$eq": ["$overall_approval_state", 1] }, 1, 0] }
            },
            "rejected_requests": {
              "$sum": { "$cond": [{ "$eq": ["$overall_approval_state", 2] }, 1, 0] }
            },
            "pending_requests": {
              "$sum": { "$cond": [{ "$eq": ["$overall_approval_state", 0] }, 1, 0] }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Lookup team details.",
        "stage": {
          "$lookup": {
            "from": "team",
            "localField": "_id",
            "foreignField": "team_id",
            "as": "team_info"
          }
        }
      },
      {
        "comment": "STEP 4: Unwind team info and calculate rates.",
        "stage": {
          "$unwind": "$team_info"
        }
      },
      {
        "comment": "STEP 5: Calculate completion and approval rates.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": "$_id",
            "team_name": "$team_info.name",
            "module_id": "$team_info.module_id",
            "total_requests": 1,
            "completed_requests": 1,
            "approved_requests": 1,
            "rejected_requests": 1,
            "pending_requests": 1,
            "completion_rate": {
              "$multiply": [
                { "$divide": ["$completed_requests", "$total_requests"] },
                100
              ]
            },
            "approval_rate": {
              "$multiply": [
                { "$divide": ["$approved_requests", "$total_requests"] },
                100
              ]
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with expertise-based level_staff configuration",
    "description": "Identify teams using expertise-based approval levels and their expertise categories.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with level_staff.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff to analyze expertise.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Filter teams with defined expertise.",
        "stage": {
          "$match": {
            "level_staff.expertise": { "$exists": true, "$ne": null, "$ne": "" }
          }
        }
      },
      {
        "comment": "STEP 4: Group by team and collect expertise info.",
        "stage": {
          "$group": {
            "_id": "$team_id",
            "team_name": { "$first": "$name" },
            "module_id": { "$first": "$module_id" },
            "module_name": { "$first": "$module_name" },
            "expertise_areas": {
              "$push": {
                "expertise_id": "$level_staff.expertise_id",
                "expertise": "$level_staff.expertise",
                "level_count": { "$size": "$level_staff.expertGroupLevel" }
              }
            },
            "total_expertise_areas": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 5: Sort by expertise area count.",
        "stage": {
          "$sort": { "total_expertise_areas": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams in organization {{organization_id}} with location-based configuration and their geographic distribution",
    "description": "Analyze teams with location settings to understand geographic team distribution.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with location data.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "location": { "$exists": true, "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind location values if it's an array.",
        "stage": {
          "$unwind": {
            "path": "$location.value",
            "preserveNullAndEmptyArrays": true
          }
        }
      },
      {
        "comment": "STEP 3: Group by location category.",
        "stage": {
          "$group": {
            "_id": {
              "location_category": "$location.location_category",
              "location_value": "$location.value"
            },
            "team_count": { "$sum": 1 },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name",
                "module_id": "$module_id"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 4: Group by category to get distribution.",
        "stage": {
          "$group": {
            "_id": "$_id.location_category",
            "total_teams": { "$sum": "$team_count" },
            "locations": {
              "$push": {
                "location_value": "$_id.location_value",
                "team_count": "$team_count",
                "teams": "$teams"
              }
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} where approval_sequence is true but sequence_staffs is empty",
    "description": "Identify configuration inconsistencies where sequential approval is enabled but no sequence is defined.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams with sequential approval enabled.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "is_approval_sequence": true
          }
        }
      },
      {
        "comment": "STEP 2: Filter teams with empty sequence_staffs.",
        "stage": {
          "$match": {
            "$or": [
              { "sequence_staffs": { "$exists": false } },
              { "sequence_staffs": { "$eq": [] } }
            ]
          }
        }
      },
      {
        "comment": "STEP 3: Project configuration issue details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "is_approval_sequence": 1,
            "has_level_staff": { "$ne": ["$level_staff", []] },
            "has_staffs": { "$ne": ["$staffs", []] },
            "response_required": 1
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams in organization {{organization_id}} with the highest number of approval sequences configured",
    "description": "Identify teams with the most complex sequential approval workflows.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with sequence_staffs.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "sequence_staffs": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Calculate sequence count and user totals.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "sequence_count": { "$size": "$sequence_staffs" },
            "total_sequence_users": {
              "$reduce": {
                "input": "$sequence_staffs",
                "initialValue": 0,
                "in": { "$add": ["$$value", { "$size": "$$this.users" }] }
              }
            },
            "sequence_details": {
              "$map": {
                "input": "$sequence_staffs",
                "as": "seq",
                "in": {
                  "sequence": "$$seq.sequence",
                  "user_count": { "$size": "$$seq.users" },
                  "approval_percent": "$$seq.approval_percent"
                }
              }
            }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by sequence count descending.",
        "stage": {
          "$sort": { "sequence_count": -1 }
        }
      },
      {
        "comment": "STEP 4: Limit to top 10 teams.",
        "stage": {
          "$limit": 10
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with both asset_tags and tags configured",
    "description": "Identify teams with comprehensive tagging for both asset management and general categorization.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with both tag types.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "asset_tags": { "$exists": true, "$ne": [] },
            "tags": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Project tag information and counts.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "asset_tags": 1,
            "tags": 1,
            "asset_tag_count": { "$size": "$asset_tags" },
            "tag_count": { "$size": "$tags" },
            "total_tags": { "$add": [{ "$size": "$asset_tags" }, { "$size": "$tags" }] }
          }
        }
      },
      {
        "comment": "STEP 3: Sort by total tag count.",
        "stage": {
          "$sort": { "total_tags": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval response time analysis for teams in organization {{organization_id}} by approval sequence",
    "description": "Analyze how long each approval sequence takes to complete across different teams.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approval requests by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "is_approval_completed": true
          }
        }
      },
      {
        "comment": "STEP 2: Unwind approval_sent_to array.",
        "stage": {
          "$unwind": "$approval_sent_to"
        }
      },
      {
        "comment": "STEP 3: Calculate response time for each approval.",
        "stage": {
          "$project": {
            "approval_request_id": 1,
            "approval_team_id": "$approval_team.team_id",
            "approval_sequence": "$approval_sent_to.approval_sequence",
            "approval_state": "$approval_sent_to.approval_state",
            "response_time_hours": {
              "$divide": [
                { "$subtract": ["$approval_sent_to.approval_date", "$approval_sent_to.approval_request_date"] },
                3600000
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 4: Group by team and sequence to get averages.",
        "stage": {
          "$group": {
            "_id": {
              "team_id": "$approval_team_id",
              "sequence": "$approval_sequence"
            },
            "avg_response_time": { "$avg": "$response_time_hours" },
            "min_response_time": { "$min": "$response_time_hours" },
            "max_response_time": { "$max": "$response_time_hours" },
            "total_approvals": { "$sum": 1 },
            "approved_count": {
              "$sum": { "$cond": [{ "$eq": ["$approval_state", 1] }, 1, 0] }
            }
          }
        }
      },
      {
        "comment": "STEP 5: Lookup team information.",
        "stage": {
          "$lookup": {
            "from": "team",
            "localField": "_id.team_id",
            "foreignField": "team_id",
            "as": "team_info"
          }
        }
      },
      {
        "comment": "STEP 6: Unwind and format results.",
        "stage": {
          "$unwind": "$team_info"
        }
      },
      {
        "comment": "STEP 7: Project final results.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": "$_id.team_id",
            "team_name": "$team_info.name",
            "module_id": "$team_info.module_id",
            "approval_sequence": "$_id.sequence",
            "avg_response_time_hours": { "$round": ["$avg_response_time", 2] },
            "min_response_time_hours": { "$round": ["$min_response_time", 2] },
            "max_response_time_hours": { "$round": ["$max_response_time", 2] },
            "total_approvals": 1,
            "approval_rate": {
              "$multiply": [
                { "$divide": ["$approved_count", "$total_approvals"] },
                100
              ]
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with events_thres_id configured and their threshold distribution",
    "description": "Analyze teams with event threshold monitoring and understand threshold usage patterns.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with event thresholds.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "events_thres_id": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Calculate threshold statistics.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "events_thres_id": 1,
            "threshold_count": { "$size": "$events_thres_id" },
            "has_business_profile": { "$ne": ["$business_hr_profile", null] }
          }
        }
      },
      {
        "comment": "STEP 3: Group by module to analyze distribution.",
        "stage": {
          "$group": {
            "_id": "$module_id",
            "module_name": { "$first": "$module_name" },
            "teams_with_thresholds": { "$sum": 1 },
            "total_thresholds": { "$sum": "$threshold_count" },
            "avg_thresholds_per_team": { "$avg": "$threshold_count" },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name",
                "threshold_count": "$threshold_count",
                "has_business_profile": "$has_business_profile"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by module_id.",
        "stage": {
          "$sort": { "_id": 1 }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams in organization {{organization_id}} with rule-based assignment and their configuration details",
    "description": "Identify teams using automated rule-based assignment and analyze their rule configurations.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with rule assignment.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "config.is_rule_assigned": true
          }
        }
      },
      {
        "comment": "STEP 2: Project rule configuration details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "group_type": 1,
            "rule_config": "$config",
            "staff_selection_type": 1,
            "is_approval_sequence": 1,
            "response_required": 1
          }
        }
      },
      {
        "comment": "STEP 3: Group by rule_type to analyze patterns.",
        "stage": {
          "$group": {
            "_id": "$rule_config.rule_type",
            "team_count": { "$sum": 1 },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name",
                "module_id": "$module_id",
                "rule_id": "$rule_config.rule_id",
                "staff_selection_type": "$staff_selection_type"
              }
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with mismatched approval configuration (sequential flag vs actual sequence data)",
    "description": "Identify teams where is_approval_sequence flag doesn't match the actual sequence configuration.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Add configuration analysis fields.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "is_approval_sequence": 1,
            "has_sequence_staffs": { "$ne": ["$sequence_staffs", []] },
            "has_level_staff": { "$ne": ["$level_staff", []] },
            "has_staffs": { "$ne": ["$staffs", []] },
            "sequence_count": { "$size": { "$ifNull": ["$sequence_staffs", []] } },
            "level_count": { "$size": { "$ifNull": ["$level_staff", []] } }
          }
        }
      },
      {
        "comment": "STEP 3: Find mismatched configurations.",
        "stage": {
          "$match": {
            "$or": [
              {
                "$and": [
                  { "is_approval_sequence": true },
                  { "has_sequence_staffs": false },
                  { "has_level_staff": false }
                ]
              },
              {
                "$and": [
                  { "is_approval_sequence": false },
                  { "has_sequence_staffs": true },
                  { "has_level_staff": false }
                ]
              }
            ]
          }
        }
      },
      {
        "comment": "STEP 4: Project mismatch details.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "is_approval_sequence": 1,
            "has_sequence_staffs": 1,
            "has_level_staff": 1,
            "has_staffs": 1,
            "mismatch_type": {
              "$cond": [
                {
                  "$and": [
                    { "$eq": ["$is_approval_sequence", true] },
                    { "$eq": ["$has_sequence_staffs", false] }
                  ]
                },
                "sequential_flag_but_no_sequence",
                "has_sequence_but_parallel_flag"
              ]
            }
          }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams in organization {{organization_id}} with notification channels configured at different approval levels",
    "description": "Analyze notification channel distribution across approval levels in teams.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with level_staff.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff array.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Analyze notification configurations.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "level": "$level_staff.expertGroupLevel.level",
            "level_id": "$level_staff.expertGroupLevel.level_id",
            "has_email_notify": { "$ne": ["$level_staff.expertGroupLevel.notify_mail", []] },
            "has_phone_notify": { "$ne": ["$level_staff.expertGroupLevel.notify_phone", []] },
            "email_count": { "$size": { "$ifNull": ["$level_staff.expertGroupLevel.notify_mail", []] } },
            "phone_count": { "$size": { "$ifNull": ["$level_staff.expertGroupLevel.notify_phone", []] } },
            "user_count": { "$size": "$level_staff.expertGroupLevel.users" }
          }
        }
      },
      {
        "comment": "STEP 5: Group by team to summarize notifications.",
        "stage": {
          "$group": {
            "_id": "$team_id",
            "team_name": { "$first": "$name" },
            "module_id": { "$first": "$module_id" },
            "total_levels": { "$sum": 1 },
            "levels_with_email": { "$sum": { "$cond": ["$has_email_notify", 1, 0] } },
            "levels_with_phone": { "$sum": { "$cond": ["$has_phone_notify", 1, 0] } },
            "total_email_addresses": { "$sum": "$email_count" },
            "total_phone_numbers": { "$sum": "$phone_count" },
            "level_details": {
              "$push": {
                "level": "$level",
                "has_email": "$has_email_notify",
                "has_phone": "$has_phone_notify",
                "email_count": "$email_count",
                "phone_count": "$phone_count"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 6: Filter teams with notification configurations.",
        "stage": {
          "$match": {
            "$or": [
              { "levels_with_email": { "$gt": 0 } },
              { "levels_with_phone": { "$gt": 0 } }
            ]
          }
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with business hour profiles and analyze their SLA coverage",
    "description": "Identify teams with business hour configurations and analyze their SLA-related settings.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with business hour profiles.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "business_hr_profile": { "$exists": true, "$ne": null }
          }
        }
      },
      {
        "comment": "STEP 2: Project business hour and SLA related fields.",
        "stage": {
          "$project": {
            "_id": 0,
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "business_hr_profile": 1,
            "response_required": 1,
            "is_add_notify_required": 1,
            "has_event_thresholds": { "$ne": ["$events_thres_id", []] },
            "staff_selection_type": 1,
            "is_approval_sequence": 1
          }
        }
      },
      {
        "comment": "STEP 3: Group by business hour profile to analyze usage.",
        "stage": {
          "$group": {
            "_id": "$business_hr_profile.profile_id",
            "profile_name": { "$first": "$business_hr_profile.name" },
            "team_count": { "$sum": 1 },
            "response_required_count": {
              "$sum": { "$cond": ["$response_required", 1, 0] }
            },
            "notification_required_count": {
              "$sum": { "$cond": ["$is_add_notify_required", 1, 0] }
            },
            "teams": {
              "$push": {
                "team_id": "$team_id",
                "name": "$name",
                "module_id": "$module_id",
                "response_required": "$response_required"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by team count to show most used profiles.",
        "stage": {
          "$sort": { "team_count": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Show teams in organization {{organization_id}} with the most complex approval structures (multiple levels and sequences)",
    "description": "Identify teams with the most complex approval configurations combining multiple levels and sequences.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false
          }
        }
      },
      {
        "comment": "STEP 2: Calculate complexity metrics.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "is_approval_sequence": 1,
            "sequence_count": { "$size": { "$ifNull": ["$sequence_staffs", []] } },
            "level_staff_count": { "$size": { "$ifNull": ["$level_staff", []] } },
            "total_approval_levels": {
              "$reduce": {
                "input": { "$ifNull": ["$level_staff", []] },
                "initialValue": 0,
                "in": { "$add": ["$$value", { "$size": "$$this.expertGroupLevel" }] }
              }
            },
            "total_users": {
              "$add": [
                { "$size": { "$ifNull": ["$owner", []] } },
                { "$size": { "$ifNull": ["$staffs", []] } },
                {
                  "$reduce": {
                    "input": { "$ifNull": ["$sequence_staffs", []] },
                    "initialValue": 0,
                    "in": { "$add": ["$$value", { "$size": "$$this.users" }] }
                  }
                },
                {
                  "$reduce": {
                    "input": { "$ifNull": ["$level_staff", []] },
                    "initialValue": 0,
                    "in": {
                      "$add": [
                        "$$value",
                        {
                          "$reduce": {
                            "input": "$$this.expertGroupLevel",
                            "initialValue": 0,
                            "in": { "$add": ["$$value", { "$size": "$$this.users" }] }
                          }
                        }
                      ]
                    }
                  }
                }
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 3: Calculate complexity score.",
        "stage": {
          "$project": {
            "team_id": 1,
            "name": 1,
            "module_id": 1,
            "module_name": 1,
            "is_approval_sequence": 1,
            "sequence_count": 1,
            "level_staff_count": 1,
            "total_approval_levels": 1,
            "total_users": 1,
            "complexity_score": {
              "$add": [
                { "$multiply": ["$sequence_count", 2] },
                { "$multiply": ["$level_staff_count", 3] },
                { "$multiply": ["$total_approval_levels", 1] },
                { "$multiply": ["$total_users", 0.1] }
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 4: Sort by complexity score descending.",
        "stage": {
          "$sort": { "complexity_score": -1 }
        }
      },
      {
        "comment": "STEP 5: Limit to top 15 most complex teams.",
        "stage": {
          "$limit": 15
        }
      }
    ]
  },
  {
    "nl_query": "Find teams in organization {{organization_id}} with duplicate users across different approval levels",
    "description": "Identify teams where the same user appears in multiple approval levels, which might indicate configuration issues.",
    "collection": "team",
    "pipeline": [
      {
        "comment": "STEP 1: Match teams by organization with level_staff.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "level_staff": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind level_staff array.",
        "stage": {
          "$unwind": "$level_staff"
        }
      },
      {
        "comment": "STEP 3: Unwind expertGroupLevel array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel"
        }
      },
      {
        "comment": "STEP 4: Unwind users array.",
        "stage": {
          "$unwind": "$level_staff.expertGroupLevel.users"
        }
      },
      {
        "comment": "STEP 5: Group by team and user to count appearances.",
        "stage": {
          "$group": {
            "_id": {
              "team_id": "$team_id",
              "user_id": "$level_staff.expertGroupLevel.users.profile_id"
            },
            "team_name": { "$first": "$name" },
            "module_id": { "$first": "$module_id" },
            "user_email": { "$first": "$level_staff.expertGroupLevel.users.email" },
            "user_name": { "$first": "$level_staff.expertGroupLevel.users.full_name" },
            "level_appearances": { "$sum": 1 },
            "levels": {
              "$push": {
                "level": "$level_staff.expertGroupLevel.level",
                "level_id": "$level_staff.expertGroupLevel.level_id",
                "expertise": "$level_staff.expertise"
              }
            }
          }
        }
      },
      {
        "comment": "STEP 6: Filter users appearing in multiple levels.",
        "stage": {
          "$match": {
            "level_appearances": { "$gt": 1 }
          }
        }
      },
      {
        "comment": "STEP 7: Group by team to show duplicate users.",
        "stage": {
          "$group": {
            "_id": "$_id.team_id",
            "team_name": { "$first": "$team_name" },
            "module_id": { "$first": "$module_id" },
            "duplicate_users": {
              "$push": {
                "user_id": "$_id.user_id",
                "user_name": "$user_name",
                "user_email": "$user_email",
                "level_appearances": "$level_appearances",
                "levels": "$levels"
              }
            },
            "total_duplicates": { "$sum": 1 }
          }
        }
      },
      {
        "comment": "STEP 8: Sort by total duplicates descending.",
        "stage": {
          "$sort": { "total_duplicates": -1 }
        }
      }
    ]
  },
  {
    "nl_query": "Show approval escalation patterns for teams in organization {{organization_id}} based on approval timeouts",
    "description": "Analyze approval escalation patterns by examining approval sequences that exceed typical response times.",
    "collection": "approval",
    "pipeline": [
      {
        "comment": "STEP 1: Match approval requests by organization.",
        "stage": {
          "$match": {
            "organization": "{{organization_id}}",
            "is_deleted": false,
            "approval_sent_to": { "$exists": true, "$ne": [] }
          }
        }
      },
      {
        "comment": "STEP 2: Unwind approval_sent_to array.",
        "stage": {
          "$unwind": "$approval_sent_to"
        }
      },
      {
        "comment": "STEP 3: Calculate response time and identify timeouts.",
        "stage": {
          "$project": {
            "approval_request_id": 1,
            "approval_team_id": "$approval_team.team_id",
            "module_id": 1,
            "approval_sequence": "$approval_sent_to.approval_sequence",
            "approval_state": "$approval_sent_to.approval_state",
            "approved_by": "$approval_sent_to.approved_by",
            "request_date": "$approval_sent_to.approval_request_date",
            "approval_date": "$approval_sent_to.approval_date",
            "response_time_hours": {
              "$divide": [
                { "$subtract": ["$approval_sent_to.approval_date", "$approval_sent_to.approval_request_date"] },
                3600000
              ]
            },
            "is_timeout": {
              "$cond": [
                { "$gt": [{ "$subtract": ["$approval_sent_to.approval_date", "$approval_sent_to.approval_request_date"] }, 172800000] },
                true,
                false
              ]
            }
          }
        }
      },
      {
        "comment": "STEP 4: Group by team to analyze timeout patterns.",
        "stage": {
          "$group": {
            "_id": "$approval_team_id",
            "total_approvals": { "$sum": 1 },
            "timeout_count": { "$sum": { "$cond": ["$is_timeout", 1, 0] } },
            "avg_response_time": { "$avg": "$response_time_hours" },
            "max_response_time": { "$max": "$response_time_hours" },
            "sequence_timeouts": {
              "$push": {
                "$cond": [
                  "$is_timeout",
                  {
                    "sequence": "$approval_sequence",
                    "response_time": "$response_time_hours",
                    "approval_state": "$approval_state"
                  },
                  "$$REMOVE"
                ]
              }
            }
          }
        }
      },
      {
        "comment": "STEP 5: Calculate timeout percentage.",
        "stage": {
          "$project": {
            "_id": 1,
            "total_approvals": 1,
            "timeout_count": 1,
            "timeout_percentage": {
              "$multiply": [
                { "$divide": ["$timeout_count", "$total_approvals"] },
                100
              ]
            },
            "avg_response_time": { "$round": ["$avg_response_time", 2] },
            "max_response_time": { "$round": ["$max_response_time", 2] },
            "sequence_timeouts": 1
          }
        }
      },
      {
        "comment": "STEP 6: Lookup team information.",
        "stage": {
          "$lookup": {
            "from": "team",
            "localField": "_id",
            "foreignField": "team_id",
            "as": "team_info"
          }
        }
      },
      {
        "comment": "STEP 7: Filter teams with timeout issues.",
        "stage": {
          "$match": {
            "timeout_percentage": { "$gt": 10 }
          }
        }
      },
      {
        "comment": "STEP 8: Sort by timeout percentage descending.",
        "stage": {
          "$sort": { "timeout_percentage": -1 }
        }
      }
    ]
  }
]
```