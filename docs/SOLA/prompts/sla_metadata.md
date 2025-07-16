# SLA Metadata
---


# Document Summary : SLA (Service Level Agreement) Profile
## Collection Name : sla_profile


```json
{
  "_id": {
    "$oid": "665f022989cb8977bea1a34f"
  },
  "sla_id": "130654389142763343872",
  "name": "RAN-SLA_old130654389142763343872",
  "agreement_type": {
    "name": "Sla",
    "id": 1
  },
  "type": {
    "id": 1,
    "name": "Service"
  },
  "status": {
    "id": 3,
    "name": "In-Active"
  },
  "status_depth": 1,
  "process_type": {
    "id": 10,
    "name": "Ticket"
  },
  "description": "",
  "scope": null,
  "start_date": {
    "$date": "2024-06-01T00:00:00.000Z"
  },
  "expiry_date": {
    "$date": "2030-06-01T00:00:00.000Z"
  },
  "next_review_date": null,
  "notification_date": null,
  "version": null,
  "compliance_target": 99,
  "owner": {},
  "approver": [],
  "reviewer": [],
  "approval_flag": 0,
  "review_flag": 0,
  "business_hr_template": {
    "name": "24*7",
    "id": "130633727582231597056"
  },
  "is_deleted": true,
  "is_pre_configured": false,
  "organization": "130633727120791048192",
  "creation_time": {
    "$date": "2024-06-04T12:01:45.446Z"
  },
  "last_updated_time": {
    "$date": "2024-10-21T06:20:34.020Z"
  },
  "priority_enabled": 1,
  "ola": null,
  "uc": null,
  "target_profile": [
    {
      "id": 2,
      "name": "RAN Response Target",
      "metric": {
        "metric_id": "130633735487924211712",
        "name": "Response",
        "id": "130633735487924211712",
        "module_id": 10
      },
      "bussiness_hour_data": {
        "id": 1,
        "name": "new_options",
        "options": {
          "business_hour": [
            {
              "id": 1,
              "severity": "Critical",
              "day": "",
              "hour": "",
              "minute": 15,
              "total_minute": 15,
              "color": "danger",
              "icon": "far fa-triangle-exclamation"
            },
            {
              "id": 2,
              "severity": "High",
              "day": "",
              "hour": "",
              "minute": 30,
              "total_minute": 30,
              "color": "warning",
              "icon": "far fa-angles-up"
            },
            {
              "id": 3,
              "severity": "Medium",
              "day": "",
              "hour": 1,
              "minute": null,
              "total_minute": null,
              "color": "info",
              "icon": "far fa-angle-up"
            },
            {
              "id": 4,
              "severity": "Low",
              "day": "",
              "hour": 4,
              "minute": null,
              "total_minute": null,
              "color": "success",
              "icon": "far fa-hyphen"
            }
          ],
          "non_critical_hour": [
            {
              "id": 1,
              "severity": "Critical",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "danger",
              "icon": "far fa-triangle-exclamation"
            },
            {
              "id": 2,
              "severity": "High",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "warning",
              "icon": "far fa-angles-up"
            },
            {
              "id": 3,
              "severity": "Medium",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "info",
              "icon": "far fa-angle-up"
            },
            {
              "id": 4,
              "severity": "Low",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "success",
              "icon": "far fa-hyphen"
            }
          ],
          "non_bussiness_hour": [
            {
              "id": 1,
              "severity": "Critical",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "danger",
              "icon": "far fa-triangle-exclamation"
            },
            {
              "id": 2,
              "severity": "High",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "warning",
              "icon": "far fa-angles-up"
            },
            {
              "id": 3,
              "severity": "Medium",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "info",
              "icon": "far fa-angle-up"
            },
            {
              "id": 4,
              "severity": "Low",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "success",
              "icon": "far fa-hyphen"
            }
          ]
        }
      },
      "is_non_biz_hr_disabled": false
    },
    {
      "id": 3,
      "name": "RAN Resolution Target",
      "metric": {
        "metric_id": "130633735492487614464",
        "name": "Resolution Metric",
        "id": "130633735492487614464",
        "module_id": 10
      },
      "bussiness_hour_data": {
        "id": 1,
        "name": "new_options",
        "options": {
          "business_hour": [
            {
              "id": 1,
              "severity": "Critical",
              "day": "",
              "hour": 4,
              "minute": "",
              "total_minute": 240,
              "color": "danger",
              "icon": "far fa-triangle-exclamation"
            },
            {
              "id": 2,
              "severity": "High",
              "day": "",
              "hour": 8,
              "minute": "",
              "total_minute": 480,
              "color": "warning",
              "icon": "far fa-angles-up"
            },
            {
              "id": 3,
              "severity": "Medium",
              "day": "",
              "hour": 10,
              "minute": "",
              "total_minute": 600,
              "color": "info",
              "icon": "far fa-angle-up"
            },
            {
              "id": 4,
              "severity": "Low",
              "day": 3,
              "hour": "",
              "minute": "",
              "total_minute": 4320,
              "color": "success",
              "icon": "far fa-hyphen"
            }
          ],
          "non_critical_hour": [
            {
              "id": 1,
              "severity": "Critical",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "danger",
              "icon": "far fa-triangle-exclamation"
            },
            {
              "id": 2,
              "severity": "High",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "warning",
              "icon": "far fa-angles-up"
            },
            {
              "id": 3,
              "severity": "Medium",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "info",
              "icon": "far fa-angle-up"
            },
            {
              "id": 4,
              "severity": "Low",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "success",
              "icon": "far fa-hyphen"
            }
          ],
          "non_bussiness_hour": [
            {
              "id": 1,
              "severity": "Critical",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "danger",
              "icon": "far fa-triangle-exclamation"
            },
            {
              "id": 2,
              "severity": "High",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "warning",
              "icon": "far fa-angles-up"
            },
            {
              "id": 3,
              "severity": "Medium",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "info",
              "icon": "far fa-angle-up"
            },
            {
              "id": 4,
              "severity": "Low",
              "day": "",
              "hour": "",
              "minute": "",
              "total_minute": 0,
              "color": "success",
              "icon": "far fa-hyphen"
            }
          ]
        }
      },
      "is_non_biz_hr_disabled": false
    }
  ],
  "applied_for": [
    {
      "serviceOption": {
        "id": "1",
        "name": "Impact Service",
        "key": "service",
        "descr": "-"
      }.,
      "id": 1,
      "conditionOption": {
        "label": "not in",
        "name": "not in",
        "descr": "-"
      },
      "value": [
        {
          "id": "668d2bd9f913059de510843b",
          "name": "Failure 40% or more",
          "service_id": "131382162331102351360",
          "service_name": "Failure 40% or more",
          "value_id": "131382162331102351360"
        },
        {
          "id": "668d2c2291cb71e8d9119e23",
          "name": "Failure less than 10%",
          "service_id": "131332756281840635904",
          "service_name": "Failure less than 10%",
          "value_id": "131332756281840635904"
        },
        {
          "id": "668d2c40aa99c3064e3a51ef",
          "name": "Failure less than 40%",
          "service_id": "131329267412797231104",
          "service_name": "Failure less than 40%",
          "value_id": "131329267412797231104"
        },
        {
          "id": "668d2c92ed89f7f4463ffd3e",
          "name": "Failure 2 or more sector",
          "service_id": "131398556964300525568",
          "service_name": "Failure 2 or more sector",
          "value_id": "131398556964300525568"
        },
        {
          "id": "668d2caf91cb71e8d9119e27",
          "name": "Failure less than 2 sector",
          "service_id": "131332793994606284800",
          "service_name": "Failure less than 2 sector",
          "value_id": "131332793994606284800"
        },
        {
          "id": "668d2d3e91cb71e8d9119e2b",
          "name": "Failure of stand by card",
          "service_id": "131332832501672448000",
          "service_name": "Failure of stand by card",
          "value_id": "131332832501672448000"
        },
        {
          "id": "668d2d6caa99c3064e3a51f3",
          "name": "Report Issue by EMS",
          "service_id": "131329348056713793536",
          "service_name": "Report Issue by EMS",
          "value_id": "131329348056713793536"
        },
        {
          "id": "668d306aed89f7f4463ffd42",
          "name": "Failure 10% or more",
          "service_id": "131398821201157558272",
          "service_name": "Failure 10% or more",
          "value_id": "131398821201157558272"
        }
      ],
      "comparison_operator": "not in",
      "operand": "service"
    }
  ],
  "daily_data": {
    "key": "daily",
    "data": null,
    "visiable": true
  },
  "weekly_data": {
    "key": "weekly",
    "data": null,
    "visiable": false
  },
  "monthly_data": {
    "key": "monthly",
    "data": null,
    "visiable": false
  }
}
```

### Key Concepts

* **Concept 1**: SLA Profile
  - the primary key is "sla_id"
  - Each sla pofile is a entitiy that describes the sla profile a for a process (change, request, release, incident)
  - The module can be differentiated by "sla_profile.process_type.name" = "ticket | change | request | release"
  - The "sla_profile.applied_for" contains the condition when the sla profile will be applied
    - The applied_for[] has multiple condition set in a array
    - for example "applied_for[0].sericeOption.name" has the service name and "applied_for[0].sericeOption.condition" has the condition
    - for example "applied_for[0].comparision_operator" and  "applied_for[0].operand" defines the type of condition
  - "sla_pofile.business_hr_template.id" contains  the business hour temple can be found in "business_hour_profiles" collection
* **Concept 2**: SLA Metric
  - Each SLA profile can have multiple metric for the sla the timings
  - "sla_profile.metric" is a array 
  - "sla_profile.metric[0].name is the name of the metric
  - "sla_profile.metric.metric_id" is ref to ducment in diff collection called "metric"
    - metric is just the condition of the sla to start and stop based on process state
  - in each metric there is a "bussiness_hour_data" that has "bussiness_hour","non_critical_hour","non_bussiness_hour"
---

# Document Summary : Metric (Metric for the sla Profile) 
## Collection Name : metric

```json
{
  "_id": {
    "$oid": "665db32d6424b28281fc2fa7"
  },
  "metric_id": "130633735487924211712",
  "name": "Response",
  "process_type": {
    "id": 10,
    "name": "Ticket"
  },
  "agreement_type": {
    "name": "Sla",
    "id": 1
  },
  "start_time_type": {
    "id": 2,
    "name": "When Start Condition Matches"
  },
  "rule_type_conditions": {
    "startConditions": [
      {
        "name": "Response start condition",
        "type": 1,
        "id": "130633735487924211713",
        "filters": [
          {
            "id": "130633735487924211714",
            "name": "Response start condition (1)",
            "type": 1,
            "logical_operator": null,
            "operand": "state",
            "comparison_operator": "in",
            "value": [
              {
                "id": 1,
                "name": "Open",
                "color_hex": "#ea5455",
                "color": "danger",
                "background_color": "rgba(172, 103, 240, 0.12)",
                "background_color_hex": "#f5edfd",
                "icon": "far fa-circle",
                "key": "Open",
                "value_id": 1
              }
            ]
          }
        ],
        "result_filters": []
      }
    ],
    "pauseConditions": [
      {
        "name": "Response pause condition",
        "type": 2,
        "id": "131399534305046695936",
        "filters": [],
        "result_filters": []
      }
    ],
    "stopConditions": [
      {
        "name": "Response close condition",
        "type": 3,
        "id": "130633735487924211716",
        "filters": [
          {
            "id": "130633735487924211717",
            "name": "Response close condition (1)",
            "type": 3,
            "logical_operator": null,
            "operand": "state",
            "comparison_operator": "in",
            "value": [
              {
                "id": 2,
                "name": "In Progress",
                "color_hex": "#ff9f43",
                "color": "warning",
                "background_color": "rgba(172, 103, 240, 0.12)",
                "background_color_hex": "#f5edfd",
                "icon": "far fa-stopwatch",
                "key": "In Progress",
                "value_id": 2
              },
              {
                "id": 4,
                "name": "Resolved",
                "color_hex": "#28c76f",
                "color": "success",
                "background_color": "rgba(172, 103, 240, 0.12)",
                "background_color_hex": "#f5edfd",
                "icon": "far fa-circle-check",
                "key": "Resolved",
                "value_id": 4
              },
              {
                "id": 5,
                "name": "Close",
                "color_hex": "#4b4b4b",
                "color": "dark",
                "background_color": "rgba(172, 103, 240, 0.12)",
                "background_color_hex": "#f5edfd",
                "icon": "far fa-check-double",
                "key": "Close",
                "value_id": 5
              }
            ]
          }
        ],
        "result_filters": []
      }
    ],
    "cancelConditions": [
      {
        "name": "Response cancel condition",
        "type": 4,
        "id": "131399534305046695937",
        "filters": [],
        "result_filters": []
      }
    ],
    "resumeConditions": [
      {
        "name": "Response resume condition",
        "type": 6,
        "id": "131399534305046695938",
        "filters": [],
        "result_filters": []
      }
    ]
  },
  "criticality": {},
  "objective": null,
  "description": "Response Metric",
  "is_deleted": false,
  "is_pre_configured": true,
  "default_metric_type": 1,
  "organization": "130633727120791048192",
  "creation_time": {
    "$date": "2024-06-03T12:12:29.590Z"
  },
  "last_updated_time": {
    "$date": "2024-07-09T13:27:39.112Z"
  }
}
```
### Key Concepts

***Concept 1**: Metric relation to sla_profile
  - the primary key is "metric_id"
  - Each metric is a entitiy that conditions for the sla to start in a process "ticket | change | request | release"
  - The module can be differentiated by "metric.process_type.name" = "ticket | change | request | release"
  - The "sla_profile.start_time_type.id" == 2  it will start based on the condition
  - The "sla_profile.rule_type_conditions" contains the condition when the sla will start, stop, pause and more
    - it has "startConditions", "pauseConditions","stopConditions", "cancelConditions","resumeConditions" as array for conditons refer to doc
  - "sla_pofile.business_hr_template.id" contains  the business hour temple can be found in "business_hour_profiles" collection
  
---

---
# Document Summary : SLM Metric Data  (SLA tracking document for A process) 
## Collection Name : slm_metric_data

```json
{
  "_id": {
    "$oid": "678e001c28c86493f6673682"
  },
  "metric_data_id": "135968178428439433216",
  "module_id": 10,
  "reference_id": "135968045696501682176",
  "sla_profile": "101967462780239876096",
  "slm_data_source_id": "135968045827766620160",
  "metric": "130633735487924211712",
  "metric_target": 60,
  "start_metric_type_id": "130633735487924211713",
  "service_id": "132412637948421345280",
  "vendor_id": "",
  "is_uc_active": false,
  "vendor_esc_status": 0,
  "impact_asset": [],
  "metric_value": 56,
  "pso_time": 0,
  "adjustment_time": 0,
  "adjustment_biz_time": 0,
  "adjustment_pso_time": 0,
  "adjustment_recalculate": 0,
  "is_freezed_adjustment": false,
  "metric_expected_time": {
    "$date": "2025-01-20T08:49:46.914Z"
  },
  "is_breached": false,
  "is_non_adjust_breached": false,
  "is_excluded": false,
  "is_target_update": false,
  "status": 3,
  "thread_status": 3,
  "threshold": 1,
  "recalculate_on_rule": 0,
  "metric_start_time": {
    "$date": "2025-01-20T07:49:46.914Z"
  },
  "metric_end_time": {
    "$date": "2025-01-20T07:50:43.435Z"
  },
  "total_hold": 0,
  "target_base_on": 3,
  "event_target_value": 1,
  "hold_start_time": null,
  "event_time": null,
  "creation_time": {
    "$date": "2025-01-20T07:49:48.260Z"
  },
  "last_calculated": {
    "$date": "2025-01-20T07:51:48.465Z"
  },
  "is_deleted": false,
  "organization": "130633727120791048192",
  "comment": null,
  "is_comment": false,
  "end_metric_type_id": "130633735487924211716"
}
```
### Key Concepts




***Concept 1**: SLM metric data (sla calculations for a specific process)
  - the "slm_metric_data.module_id" define the module it is for like incident = 10
  - the primary key is "metric_data_id"
  - "slm_metric_data.module_id" is the reference to the process document to which its attached
  - there will be multiple docs here for each process that is number of metrics added to the sla_profile
  - "slm_metric_data.sla_profile" is the sla_profile from which the data is calculated
  - there are many feilds here that you can read and lead what they mean for the sla (service level agreement)
  - this also has some important keys like "is_breached","pso_time"(planned service outage),"metric_start_time","metric_end_time"
  - you can find more once you analyze it
  
---
---
# Document Summary : SLM Metric Monitor Event  (SLA Metric event tracing doc) 
## Collection Name : slm_metric_monitor_event

```json
{
  "_id": {
    "$oid": "678e001c28c86493f6673683"
  },
  "metric_event_id": "135968178430586916864",
  "module_id": 10,
  "reference_id": "135968045696501682176",
  "sla_profile": "101967462780239876096",
  "metric": "130633735487924211712",
  "metric_data_id": "135968178428439433216",
  "slm_data_source_id": "135968045827766620160",
  "group_id": "133771471523643658240",
  "level_id": "",
  "sub_group_id": "",
  "group_type": "Operation and Technical",
  "vendor_id": "",
  "start_time": {
    "$date": "2025-01-20T07:49:46.914Z"
  },
  "end_time": {
    "$date": "2025-01-20T07:50:43.435Z"
  },
  "start_event_data": {
    "service": "132412637948421345280",
    "requester": "130675316420842098688",
    "requester_tag": "",
    "asset_tags": "",
    "priority": 3,
    "severity": "",
    "urgency": "",
    "impact": "",
    "incident_source": 1,
    "incident_type": 1,
    "device_type": "",
    "creation_time": {
      "$date": "2025-01-20T07:49:46.914Z"
    },
    "last_update_time": {
      "$date": "2025-01-20T07:49:46.914Z"
    },
    "state": 1,
    "impacted_asset": [],
    "actual_start_date": "",
    "status": {
      "name": "New",
      "state": "UI.k_open",
      "state_id": 1,
      "guid": "wf_k55u40yhyhn_it4x49atk6e",
      "id": "113408908852133368808",
      "color": "#ea5455",
      "background_color": "#f5edfd",
      "workflow_id": "130633734307613511680",
      "is_response_time": false,
      "workflow_name": "Ticket Workflow",
      "is_start": true,
      "transition_state": []
    },
    "group": "133771471523643658240",
    "group_type": "Operation and Technical",
    "level": 1,
    "expertise": "133651255584449630208",
    "modify_flag": [],
    "user_tag_id": [
      "132863374127476510720"
    ],
    "asset_tag_id": ""
  },
  "end_event_data": {
    "service": "132412637948421345280",
    "requester": "130675316420842098688",
    "requester_tag": "",
    "asset_tags": "",
    "priority": 3,
    "severity": "",
    "urgency": "",
    "impact": "",
    "incident_source": 1,
    "incident_type": 1,
    "device_type": "",
    "creation_time": {
      "$date": "2025-01-20T07:49:46.914Z"
    },
    "last_update_time": {
      "$date": "2025-01-20T07:50:43.435Z"
    },
    "state": 2,
    "impacted_asset": [],
    "actual_start_date": "",
    "status": {
      "name": "Analysis",
      "state": "UI.k_In_progress",
      "state_id": 2,
      "guid": "wf_tutesg8dkoq_b2wgvtkogjj",
      "color": "#ff9f43",
      "background_color": "rgba(172, 103, 240, 0.12)",
      "id": "113409077653206471656"
    },
    "group": "133771471523643658240",
    "group_type": "Operation and Technical",
    "level": 1,
    "expertise": "133651255584449630208",
    "modify_flag": [],
    "user_tag_id": [
      "132863374127476510720"
    ],
    "asset_tag_id": ""
  },
  "event_type_flag": 1,
  "event_type": [],
  "event_status": 3,
  "thread_status": 2,
  "biz_time": {
    "$numberLong": "56"
  },
  "non_biz_time": {
    "$numberLong": "0"
  },
  "biz_excluding_hrs": {
    "$numberLong": "0"
  },
  "biz_pso_hrs": 0,
  "pso_start_time": null,
  "pso_end_time": null,
  "adjustment_pso_time": 0,
  "cal_time": 56,
  "value": 0,
  "is_active": false,
  "is_deleted": false,
  "last_calculated": {
    "$date": "2025-01-21T07:56:27.626Z"
  },
  "creation_time": {
    "$date": "2025-01-20T07:49:48.260Z"
  },
  "organization": "130633727120791048192"
}
```
### Key Concepts




***Concept 1**: SLM metric monitor event
  - this document contains the sla time details for event whenever the metric starts or stops based on the condition of the "metric" defined in sla metric
  - Each "slm_metric_monitor_event.metric_data_id" is the reference to "slm_metric_data" document
  - There could be multiple docs for single metric data as there are multiple conditions in a metric
  - This also has the ref to metric templte documents in "metric" key
  - "start_event_data" and "end_event_data" contains the event details
  ---
  ---
Sure! Based on the documents you've shared, here's a refined and AI-ready version of the **Relationships**, **Interpretation**, **Tags**, and **Notes** sections:

---

## Relationships Between Documents

* **`sla_profile` ↔ `metric`**

  * Each SLA profile includes one or more SLA metrics (in `target_profile.metric`).
  * Each metric defines the start, pause, resume, and stop conditions using `rule_type_conditions`.
  * The linkage happens through `metric.metric_id`, which connects to the `metric` collection via `sla_profile.target_profile[].metric.metric_id`.

* **`metric` ↔ `slm_metric_data`**

  * `slm_metric_data.metric` references a metric that defines how SLA tracking is configured.
  * This document tracks the actual SLA performance for a specific ticket, request, etc.

* **`sla_profile` ↔ `slm_metric_data`**

  * `slm_metric_data.sla_profile` points to the SLA profile used to calculate performance data.

* **`slm_metric_data` ↔ `slm_metric_monitor_event`**

  * `slm_metric_monitor_event.metric_data_id` maps to the corresponding `slm_metric_data.metric_data_id`.
  * This allows tracking of event-wise metric transitions (e.g., when SLA timer started/stopped and why).

* **`slm_metric_monitor_event` ↔ `metric`**

  * Contains `metric` reference to indicate which metric triggered the event.

---




## 

`#sla` `#service-outage` `#breached` `#active` `#active sla`
`#running sla` `#breached` `#monitor event` `#states` `#document-linking` `#slm`

---



