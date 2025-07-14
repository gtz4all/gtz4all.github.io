---
title: "Managing Multiple Github Accounts"
description: "Cloud Computing Documentation"
---

# GitHub Actions External Triggers

## GitHub Action Triggers

* `workflow_dispatch`
* `repository_dispatch`

> The purpose of this document is to solve a specific problem. However, the above flow can be used to solve other problems.

---

## `workflow_dispatch`

The following GitHub App trigger requires variables to be defined in the workflow.

* Variables can be string or boolean.
* Multiple variables can be passed as strings. Additional work is required to access them as JSON variables.

### Sample Workflow

```yaml
name: Workflow Dispatch

on:
  workflow_dispatch:
    inputs:
      variable1:
        description: 'Request'

jobs:
  build:
    name: Start Build
    runs-on: ubuntu-latest
    steps:
      - name: Step1
        run: |
          echo Expose all Variables
          export

      - name: Step2
        run: |
          echo Expose client payload field
          echo "field: ${{ github.event.inputs.variable1 }}"

      - name: Step3
        run: |
          echo Expose client_payload
          echo "payload: ${{ toJson(github.event.inputs) }}"
```

### Trigger with `curl`

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token <github_pat_token>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/KG48037/gcp_notes/actions/workflows/workflow_dispatch.yml/dispatches \
  -d '{"ref":"main","inputs":{"variable1":"{\"var1\": 1, \"var2\": 2, \"var3\": 3}"}}'
```

---

## `repository_dispatch`

* JSON variables can be accessed via the `client_payload`.

📄 [GitHub Docs - Workflow Dispatch Events and Payloads](https://docs.github.com/en/webhooks-and-events/webhooks/webhook-events-and-payloads#workflow_dispatch)

### Sample Workflow

```yaml
name: Build Request

on:
  repository_dispatch:
    types: build

jobs:
  build:
    name: Start Build
    runs-on: ubuntu-latest
    steps:
      - name: Step1
        run: |
          echo Expose all Variables
          export

      - name: Step2
        run: |
          echo Expose client payload field
          echo "field: ${{ github.event.client_payload.variable1 }}"

      - name: Step3
        run: |
          echo Expose client_payload
          echo "payload: ${{ toJson(github.event.client_payload) }}"
```

### Trigger with `curl`

```bash
curl -H "Accept: application/vnd.github.everest-preview+json" \
     -H "Authorization: token <github_pat_token>" \
     --request POST \
     --data '{
       "event_type": "build",
       "client_payload": {
         "variable1": "lab",
         "variable2": "sandbox"
       }
     }' \
     https://api.github.com/repos/KG48037/gcp_notes/dispatches
```

---

## JIRA Project Automation Integration

This example uses **Jira Cloud** to demonstrate the use of Jira Project Automation with `repository_dispatch`.

### Create an Automation Rule

#### Rule Details

| Component | Description   |
| --------- | ------------- |
| Trigger   | Event Trigger |
| Action    | GitHub Action |

---

### GitHub Action

```yaml
name: Webhook Request

on:
  repository_dispatch:
    types: [webhook, jira]

jobs:
  build:
    name: Start Request
    runs-on: ubuntu-latest
    steps:
      - name: Step1
        run: |
          echo Expose all Variables
          export

      - name: Step2
        run: |
          echo Expose client_payload
          echo "payload: ${{ toJson(github.event.client_payload) }}"
```

---

### Jira Action Configuration

* **Web request URL:**

```
https://api.github.com/repos/KG48037/gcp_notes/dispatches
```

* **HTTP method:** `POST`

* **Web request body:** Custom data

```json
{
  "event_type": "webhook",
  "client_payload": {
    "IssueName": "{{issue.key}}",
    "IssueDescription": "{{issue.fields.description}}",
    "IssueDesc": "{{issue.description}}",
    "IssueProject": "{{issue.project}}",
    "IssueStatus": "{{issue.status}}",
    "IssueStatusName": "{{issue.status.name}}",
    "IssueSummary": "{{issue.summary}}",
    "IssueVersion": "{{issue.versions}}"
  }
}
```

---

Let me know if you'd like this exported to a `.md` file.
