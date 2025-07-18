#NLP

```python

# Extract enhanced metadata for each collection
collection_metadata = {
    "collection_name": "incident",
    "description": "Incident tickets with SLA tracking",
    "key_fields": {
        "display_id": "Human-readable incident ID",
        "basic_info.state.name": "Current incident state",
        "creation_time": "When incident was created",
        "current_assignment_info.assignee_profile.full_name": "Assigned engineer"
    },
    "common_queries": ["find my incidents", "incidents about to breach", "overdue tickets"],
    "relationships": ["links to slm_metric_data via reference_id"]
}
```

```python
templates = {
    "my_incidents": {
        "intent": "find incidents assigned to user",
        "template": "db.incident.find({'current_assignment_info.assignee_profile.full_name': '{{user_name}}'})",
        "parameters": ["user_name"]
    },
    "breach_analysis": {
        "intent": "find incidents about to breach SLA",
        "template": "db.slm_metric_data.find({'is_breached': false, 'metric_expected_time': {'$lt': new Date(Date.now() + {{hours}}*3600000)}})",
        "parameters": ["hours"]
    }
}
```

```python
# Minimal context prompting approach
system_prompt = """You are a MongoDB query generator for SLA management.
Given: user intent, relevant schema fields, and query template
Generate: Valid MongoDB aggregation pipeline

Schema Context: {minimal_schema}
Template: {relevant_template}
User Query: {user_query}

Output only valid MongoDB JSON - no explanations."""
```
