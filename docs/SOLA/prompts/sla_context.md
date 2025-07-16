# SLA Short Context
```json
{
  "schema_description": "This document outlines the schema for four MongoDB collections used for Service Level Agreement (SLA) management. Use this to construct aggregation pipelines.",
  "collections": {
    "slm_metric_data": {
      "description": "The main transactional collection. Each document represents a single SLA metric being tracked against a specific record (like a ticket or change). This is the primary collection for most analytics queries.",
      "key_fields": [
        { "field": "metric_data_id", "type": "string", "description": "Unique identifier for this specific metric instance." },
        { "field": "sla_profile", "type": "string", "description": "The ID of the SLA profile being applied. Links to the 'sla_profile' collection via 'sla_id'." },
        { "field": "metric", "type": "string", "description": "The ID of the metric template being used. Links to the 'metric' collection via 'metric_id'." },
        { "field": "module_id", "type": "string", "description": "Crucial field identifying the entity type. Common values are 'ticket' (for incidents/problems), 'change', 'request', and 'release'." },
        { "field": "service_id", "type": "string", "description": "Identifier for the business service this SLA applies to." },
        { "field": "is_breached", "type": "boolean", "description": "True if the SLA for this metric instance was violated." },
        { "field": "status", "type": "integer", "description": "The current status of the metric (1: Active, 2: Paused, 3: Completed)." },
        { "field": "metric_start_time", "type": "datetime", "description": "When the metric clock started." },
        { "field": "metric_end_time", "type": "datetime", "description": "When the metric clock stopped (upon completion)." },
        { "field": "metric_expected_time", "type": "datetime", "description": "The deadline by which the metric should be completed to avoid a breach." },
        { "field": "total_hold", "type": "long", "description": "Total time in milliseconds the metric was paused or on hold." }
      ]
    },
    "sla_profile": {
      "description": "Defines the high-level SLA profiles, such as 'Gold Support' or 'Standard Change'. It acts as a container for specific metrics and targets.",
      "key_fields": [
        { "field": "sla_id", "type": "string", "description": "Primary key for an SLA profile. Referenced by 'slm_metric_data.sla_profile'." },
        { "field": "name", "type": "string", "description": "User-friendly name of the SLA profile (e.g., '24x7 Critical Support')." },
        { "field": "compliance_target", "type": "double", "description": "The overall compliance percentage this profile aims for (e.g., 99.5)." }
      ]
    },
    "metric": {
      "description": "Defines the templates for individual metrics, such as 'Time to Resolution' or 'Time to First Response'.",
      "key_fields": [
        { "field": "metric_id", "type": "string", "description": "Primary key for a metric template. Referenced by 'slm_metric_data.metric'." },
        { "field": "name", "type": "string", "description": "User-friendly name of the metric (e.g., 'Incident Resolution Time')." }
      ]
    },
    "slm_metric_monitor_event": {
      "description": "A detailed event log for each metric instance. It records every start, stop, pause, and resume event.",
      "key_fields": [
        { "field": "metric_data_id", "type": "string", "description": "Links each event back to a specific metric instance in 'slm_metric_data'." },
        { "field": "event_type", "type": "string", "description": "The type of event (e.g., 'START', 'STOP', 'PAUSE')." },
        { "field": "start_time", "type": "datetime", "description": "The timestamp of the event." },
        { "field": "biz_time", "type": "long", "description": "Time elapsed during business hours for this event." },
        { "field": "non_biz_time", "type": "long", "description": "Time elapsed outside business hours for this event." }
      ]
    }
  },
  "relationships": [
    {
      "from": "slm_metric_data",
      "to": "sla_profile",
      "join_type": "$lookup",
      "description": "To get the SLA profile name, join 'slm_metric_data' with 'sla_profile'.",
      "syntax": {
        "$lookup": {
          "from": "sla_profile",
          "localField": "sla_profile",
          "foreignField": "sla_id",
          "as": "profile_info"
        }
      }
    },
    {
      "from": "slm_metric_data",
      "to": "metric",
      "join_type": "$lookup",
      "description": "To get the metric template name, join 'slm_metric_data' with 'metric'.",
      "syntax": {
        "$lookup": {
          "from": "metric",
          "localField": "metric",
          "foreignField": "metric_id",
          "as": "metric_info"
        }
      }
    },
    {
      "from": "slm_metric_data",
      "to": "slm_metric_monitor_event",
      "join_type": "$lookup",
      "description": "To analyze the detailed event history of a metric, join 'slm_metric_data' with 'slm_metric_monitor_event'.",
      "syntax": {
        "$lookup": {
          "from": "slm_metric_monitor_event",
          "localField": "metric_data_id",
          "foreignField": "metric_data_id",
          "as": "events"
        }
      }
    }
  ]
}