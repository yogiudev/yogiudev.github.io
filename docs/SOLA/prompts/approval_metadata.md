# Approval Metadata

---

# Document Summary : Team Approval

## Collection Name : approval

This document captures approval requests generated within workflow-driven modules (like Incident, Request, Problem3. 👤 Team Ownership
`approval_team.team_id`: Links to a team in the teams collection

This team's `sequence_staffs` defines the approval path Change). It tracks the approval trail, team configurations, current status, and individual responses across sequences.

```json
{
  "_id": {
    "$oid": "66bf4aafa5596dca65ef4510"
  },
  "approval_request_id": "130929801089699024896",
  "module_id": 47,
  "ref_id": "132349958401247154176",
  "organization": "130633727120791048192",
  "approval_team": {
    "team_id": "131742700764457340928"
  },
  "is_approved": false,
  "submitted_by": "25",
  "approval_sent_to": [
    {
      "approval_data_id": "130929801090504331264",
      "approved_by": {
        "profile_id": "130633727402648276992",
        "email": "cnops.nocsupport@tcs.com",
        "full_name": "TCS ITSM",
        "sequence": 1
      },
      "approved_on_behalf": {},
      "approval_source": 1,
      "approval_request_date": {
        "$date": "2024-08-16T12:48:47.359Z"
      },
      "approval_sequence": 1,
      "approval_state": 0
    }
  ],
  "previous_status": {
    "name": "Cab Approval",
    "state": "UI.k_approval",
    "state_id": 6,
    "guid": "wf_77qdu2n72vc_s9by4gn7k3l",
    "color": "#4b4b4b",
    "background_color": "rgba(172, 103, 240, 0.12)",
    "id": "115885640987024822278",
    "workflow_id": "132156280567254487040",
    "workflow_name": "Major Change Workflow",
    "transition_state": []
  },
  "is_deleted": false,
  "is_active": true,
  "overall_approval_state": 0,
  "approval_sequence": 1,
  "approval_type": 1,
  "is_approval_completed": false,
  "response_required": false,
  "approval_block_flow_id": "wf_77qdu2n72vc_s9by4gn7k3l",
  "last_updated": {
    "$date": "2024-08-16T12:48:47.356Z"
  },
  "creation_date": {
    "$date": "2024-08-16T12:48:47.356Z"
  }
}
```

## 🧠 Key Concepts

1. 🧩 Module Reference
   `module_id` links this approval to a specific type of entity:
   "approval.module_id"

- 10 → Incident
- 42 → Request
- 47 → Change

`ref_id` refers to the specific record ID in that module

2. 🔁 Approval Sequence
   `approval_sequence` tracks which stage is currently being handled

`approval_sent_to` stores each approver's status:

- `sequence` helps identify order
- `approval_state`:
  - 0 = Pending
  - 1 = Approved
  - 2 = Rejected

3. 👤 Team Ownership
   approval_team.team_id: Links to a team in the teams collection

This team’s sequence_staffs defines the approval path

4. 🟢 Status Tracking
   `overall_approval_state`: Reflects final result (approved/rejected/pending)

`is_approval_completed`: Indicates if the workflow node is fully passed

5. 🛠 Workflow Integration
   `previous_status`: Carries information about the node prior to entering approval

`approval_block_flow_id`: Identifies the specific workflow block

`response_required`: Determines if each stage must receive a response

## 🔑 Primary Key

`approval_request_id` (string): Uniquely identifies each approval instance
