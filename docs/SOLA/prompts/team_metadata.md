
# Teams Collection Metadata

This document represents a Team Configuration for different modules (like incident, request, or change) within an organization. It defines team composition, approval workflows, notification mechanisms, and hierarchical approval levels.

## Collection Overview
- **Collection Name**: `teams`
- **Purpose**: Defines team structures, approval workflows, and member assignments
- **Scope**: Cross-module team management for incidents, requests, and changes

---

## Schema Definition

```json
{
  "label": "Infraon - Teams",
  "kind": "node",
  "type": "teams",
  "required": true,
  "condition": [],
  "show": [],
  "named_values": {
    "team_id": {
      "type": "string",
      "description": "Unique identifier for the team"
    },
    "name": {
      "type": "string",
      "description": "Name of the team"
    },
    "description": {
      "type": "string",
      "description": "Short description of the team's role or function"
    },
    "module_id": {
      "type": "number",
      "description": "Module the team is associated with (10=Incident, 42=Request, 47=Change)"
    },
    "module_name": {
      "type": "string",
      "description": "Name of the module corresponding to module_id"
    },
    "group_id": {
      "type": "number",
      "description": "Optional grouping ID (e.g., department or category reference)"
    },
    "group_type": {
      "type": "string",
      "description": "Type of grouping, such as Department, Location, or Function"
    },
    "staff_selection_type": {
      "type": "string",
      "enum": ["individual", "group"],
      "description": "Defines how staff are assigned (individual-based or group-based)"
    },
    "owner": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "profile_id": { "type": "string" },
          "email": { "type": "string" },
          "full_name": { "type": "string" }
        }
      },
      "description": "List of owners responsible for the team"
    },
    "staffs": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "profile_id": { "type": "string" },
          "email": { "type": "string" },
          "full_name": { "type": "string" }
        }
      },
      "description": "List of static team members when staff selection is manual"
    },
    "sequence_staffs": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "sequence": { "type": "number" },
          "approval_percent": { "type": "number" },
          "users": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "profile_id": { "type": "string" },
                "email": { "type": "string" },
                "full_name": { "type": "string" }
              }
            }
          }
        }
      },
      "description": "Approval workflow defined by sequence with approval thresholds"
    },
    "level_staff": {
      "type": "array",
      "description": "Defines hierarchical levels of approvers (used when sequence is disabled)",
      "items": {
        "type": "object",
        "properties": {
          "expertise_id": { "type": "string" },
          "expertise": { "type": "string" },
          "expertGroupLevel": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "level": { "type": "string" },
                "level_id": { "type": "number" },
                "approval_percent": { "type": "number" },
                "users": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "profile_id": { "type": "string" },
                      "email": { "type": "string" },
                      "full_name": { "type": "string" }
                    }
                  }
                },
                "notify_mail": {
                  "type": "array",
                  "items": { "type": "string" }
                },
                "notify_phone": {
                  "type": "array",
                  "items": { "type": "string" }
                }
              }
            }
          }
        }
      }
    },
    "is_approval_sequence": {
      "type": "boolean",
      "description": "If true, uses sequence-based approval (overrides level-based flow)"
    },
    "response_required": {
      "type": "boolean",
      "description": "Specifies whether this team is required to respond during the workflow"
    },
    "is_add_notify_required": {
      "type": "boolean",
      "description": "Whether additional notifications are to be triggered"
    },
    "config": {
      "type": "object",
      "description": "Optional assignment rule configuration for automation or policy",
      "properties": {
        "rule_id": { "type": "number" },
        "rule_type": { "type": "string" },
        "is_rule_assigned": { "type": "boolean" }
      }
    },
    "organization": {
      "type": "string",
      "description": "Organization or tenant identifier"
    },
    "business_hr_profile": {
      "type": "object",
      "description": "Business hours profile used for SLA alignment and escalation",
      "properties": {
        "profile_id": { "type": "string" },
        "name": { "type": "string" }
      }
    },
    "location": {
      "type": "object",
      "description": "Location metadata applicable for the team",
      "properties": {
        "location_category": { "type": "string" },
        "value": {
          "type": "array",
          "items": { "type": "string" }
        }
      }
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Optional tags for filtering or grouping teams"
    },
    "asset_tags": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Tags linked to asset ownership or scope"
    },
    "events_thres_id": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Linked event threshold rules for notifications or metrics"
    },
    "is_deleted": {
      "type": "boolean",
      "description": "Soft delete flag; true if team is logically removed"
    },
    "is_preconfigure": {
      "type": "boolean",
      "description": "Indicates if this is a preconfigured/system-generated team"
    },
    "creation_time": {
      "type": "datetime",
      "description": "Timestamp when the team was created"
    },
    "last_update_time": {
      "type": "datetime",
      "description": "Timestamp of the most recent update"
    }
  }
}
```

## 🔑 Primary Key
- `team_id` (String): Unique identifier for each team.

---

## 🧠 Key Concepts

### 1. **Module Association**
- `module_id`: Numerical identifier for the module the team belongs to.
- `module_name`: Human-readable name for the module.
  - Example mappings:
    - `10` → Incident
    - `42` → Request
    - `47` → Change

---

### 2. **Team Identity**
- `name`: Name of the team.
- `description`: Description of the team’s function or role.
- `group_type`: Type or category of the group.
- `group_id`: Reference ID to a group configuration, if any.

---

### 3. **Ownership**
- `owner`: List of users who are owners or maintainers of this team.
  ```json
  [
    {
      "profile_id": "128265819283744886784",
      "email": "hrhr@mail.com",
      "full_name": "Harish"
    }
  ]
# 👥 `staffs` vs `sequence_staffs` in `teams` Collection

These fields define the team members involved in approvals, actions, or notifications within a module. The choice between them depends on whether approvals are **parallel** or **sequential**.

---

## ✅ `staffs`
**Used when:** `is_approval_sequence = false`  
**Meaning:** All members listed are considered **equal-level participants**. Actions (like approvals or assignments) can be performed by **any or all**.

### 📌 Example
```json
"staffs": [
  {
    "profile_id": "124894839217487384",
    "email": "aashu@example.com",
    "full_name": "Aashutosh Kumar"
  },
  {
    "profile_id": "987498748937490872",
    "email": "neha@example.com",
    "full_name": "Neha Singh"
  }
]
```

# 🔁 `sequence_staffs` Format in `teams` Collection

The `sequence_staffs` field defines a **step-wise (sequential)** approval or action path, where **each stage (sequence)** must be completed before the next begins.

---

## 📘 Field: `sequence_staffs`

### ✅ Used When
- `is_approval_sequence = true`
- Approvals must follow an **ordered path**, one after another

---

## 📌 JSON Example

```json
"sequence_staffs": [
  {
    "sequence": "Sequence 1",
    "users": [
      {
        "profile_id": "111111111111111111111",
        "email": "alice.doe@example.com",
        "full_name": "Alice Doe"
      }
    ]
  },
  {
    "sequence": "Sequence 2",
    "users": [
      {
        "profile_id": "222222222222222222222",
        "email": "bob.smith@example.com",
        "full_name": "Bob Smith"
      }
    ]
  }
]