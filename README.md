# Revenue Signal & Lead Intelligence Automation

An n8n-based workflow that converts customer activity into structured revenue signals, calculates a rule-based priority score, and routes accounts to different actions.

## Business Problem

Sales teams receive customer activity signals but may struggle to identify which accounts need immediate attention.

This project demonstrates how AI-assisted signal extraction and deterministic business rules can help prioritize account activity and trigger appropriate follow-up actions.

## What It Does

1. Receives customer activity through an n8n Webhook.
2. Uses Gemini to extract structured engagement, renewal, expansion, and risk signals.
3. Validates and normalizes the extracted data using JavaScript.
4. Calculates a weighted score and assigns a priority.
5. Stores event information in PostgreSQL.
6. Routes high-priority events to an urgent-task mock action and other events to a signal-update mock action.
7. Updates the event status after a successful action.

## Architecture

Webhook → Gemini Signal Extraction → Validate & Normalize → Calculate Score & Priority → PostgreSQL → Priority Routing → Mock HTTP Action → PostgreSQL Status Update

## Technology Stack

- **n8n:** Workflow orchestration
- **Google Gemini:** Customer signal extraction
- **JavaScript:** Validation, normalization, scoring, and routing logic
- **PostgreSQL (Neon):** Event persistence and status tracking
- **HTTP Request:** Mock downstream CRM actions
- **GitHub:** Version control and project documentation

## Scoring Logic

The workflow uses illustrative weights:

| Signal | Weight |
|---|---:|
| Engagement | 25 |
| Renewal | 30 |
| Expansion | 30 |
| Risk | 15 |
| **Total** | **100** |

Signal values:

- HIGH = 1
- MEDIUM = 0.5
- LOW = 0
- UNKNOWN = 0

Each signal value is multiplied by its weight. The weighted values are added to calculate the total score.

Priority thresholds:

- HIGH: score ≥ 70
- MEDIUM: score ≥ 40 and < 70
- LOW: score < 40

These weights and thresholds are demonstration rules, not empirically validated business thresholds.

## Example

Sample input:

```json
{
  "event_id": "EVT-1001",
  "event_type": "customer_activity",
  "account_id": "ACC-1001",
  "account_name": "Acme Corp",
  "event_text": "Acme opened the pricing page four times this week. Their contract renewal is in 45 days. They are also evaluating additional users."
}
```

Illustrative extracted signals:

- Engagement: HIGH
- Renewal: HIGH
- Expansion: HIGH
- Risk: UNKNOWN

Expected score: 85/100

Expected priority: HIGH

The workflow routes this event to the urgent-task mock action.

## Reliability and Design Considerations

- AI extracts signals; deterministic JavaScript calculates the score and priority.
- Validation helps prevent unexpected model output from flowing unchecked through the workflow.
- PostgreSQL stores event information and supports duplicate-event protection using a unique event ID.
- The event ID should remain stable when the same event is retried.
- An event ID identifies an event, not an account. A single account can generate multiple events.

## Current Limitations

- The downstream CRM actions use a mock HTTP endpoint; a real CRM integration is not implemented.
- Scoring weights and priority thresholds require validation against real business outcomes.
- UNKNOWN signals currently contribute zero points, which may understate risk when information is missing.
- Failure recovery, retry behavior, and recovery of stuck PENDING events require further hardening.
- Database duplicate protection alone does not guarantee exactly-once execution of external actions.
- Production use would require stronger monitoring, security controls, and tested failure-handling paths.

## Testing

The intended test cases include:

1. A high-signal account produces a HIGH priority.
2. A low-signal account follows the non-HIGH branch.
3. Replaying the same event does not create a duplicate database record or repeat an action.
4. A new event for an existing account is stored separately.
5. A failed downstream request does not incorrectly mark the action as completed.

Test results should be documented only after each case has actually been run.

## Setup

1. Import the workflow JSON into an n8n instance.
2. Configure the required Gemini and PostgreSQL credentials inside n8n.
3. Create the required PostgreSQL table using the SQL provided with the project.
4. Configure the mock HTTP endpoint.
5. Review the node configuration and test with the sample input.
6. Run the test cases and inspect both workflow execution results and database records.

Never commit API keys, database passwords, connection strings, or other secrets to the repository.

## Learning Objectives

This project explores practical AI automation concepts, including:

- LLM-based structured extraction
- Data validation and normalization
- Deterministic business rules
- Workflow branching
- API integration
- PostgreSQL persistence
- Duplicate-event handling
- Testing and reliability considerations

## Disclaimer

This is a learning and portfolio project. The workflow demonstrates a potential business use case; it has not been validated against production sales data and should not be described as production-ready.

## Future Improvements

- Implement and test robust retry and failure-recovery paths.
- Add reliable recovery for events left in PENDING status.
- Validate scoring thresholds using labelled business data.
- Integrate a real CRM with appropriate authentication and permissions.
- Add operational monitoring and audit reporting.
