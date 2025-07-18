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
  "_id": {
    "$oid": "67802cd7031db67d6186e859"
  },
  "team_id": "117625570600208568320",
  "module_id": 53,
  "module_name": "release",
  "group_type": "Approval",
  "group_id": 1,
  "name": "North CORE Product_RM",
  "description": "North CORE Support_RM - Release and Patches",
  "owner": [
    {
      "profile_id": "130675316407151890432",
      "email": "vijay.garg@tcs.com",
      "full_name": "Vijay Garg"
    }
  ],
  "staff_selection_type": "individual",
  "is_approval_sequence": true,
  "tags": [],
  "staffs": [
    {
      "profile_id": "130675316407151890432",
      "email": "vijay.garg@tcs.com",
      "full_name": "Vijay Garg"
    }
  ],
  "sequence_staffs": [
    {
      "sequence": "Sequence 1",
      "users": [
        {
          "profile_id": "130675316407151890432",
          "email": "vijay.garg@tcs.com",
          "full_name": "Vijay Garg"
        }
      ]
    }
  ],
  "level_staff": [],
  "is_preconfigure": false,
  "is_deleted": true,
  "organization": "130633727120791048192",
  "creation_time": {
    "$date": "2025-01-09T20:08:55.848Z"
  },
  "last_update_time": {
    "$date": "2025-01-09T20:14:40.246Z"
  },
  "response_required": false,
  "approval_percentage": 10,
  "department": "133763812983579873280",
  "business_hr_profile": {
    "profile_id": "130633727582231597056",
    "name": "24*7"
  },
  "asset_tags": [],
  "is_add_notify_required": false,
  "is_demo_data": false,
  "events_thres_id": [],
  "location": {}
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
  ```

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
```
