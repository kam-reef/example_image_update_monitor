Below is a ready‑to‑drop‑in **`README.md`** that explains what this workflow does, how and when it runs, and what artifacts/notifications it produces.

***

# Check Base Image Updates Workflow

This repository includes a GitHub Actions workflow that automatically monitors a container base image for updates and notifies maintainers when a change occurs.

## Overview

The **Check Base Image Updates** workflow tracks the image digest of:

    mcr.microsoft.com/dotnet/sdk:10.0

On a scheduled basis (or on demand), the workflow:

1.  Fetches the current image digest from Microsoft's container registry
2.  Compares it to the last recorded digest stored in the repository
3.  Detects changes or responds to a forced update trigger
4.  Records the new digest in version control
5.  Creates a GitHub Issue to notify maintainers when an update is detected

This enables teams to stay informed when upstream base images change, supporting timely security reviews and rebuilds.

***

## Triggers

The workflow runs in two ways:

### 1. Scheduled Run

```yaml
schedule:
  - cron: '0 13 * * 1-5'
```

*   Runs **weekdays at 13:00 UTC (08:00 EST)**
*   Ensures regular monitoring without manual intervention

### 2. Manual Trigger

```yaml
workflow_dispatch:
  inputs:
    force_update:
      type: boolean
```

*   Can be triggered manually from the GitHub UI
*   Optional `force_update` flag allows bypassing digest comparison logic

***

## Permissions

The workflow requires the following repository permissions:

| Permission             | Purpose                                |
| ---------------------- | -------------------------------------- |
| `contents: write`      | Commit and push `digest.txt`           |
| `issues: write`        | Create GitHub Issues for notifications |
| `pull-requests: write` | Reserved for future PR automation      |

***

## Job: `check-image`

### Runner

*   Executes on a **Linux x64 runner**
*   (`self-hosted` in the current configuration)

### Timeout

*   Maximum runtime: **30 minutes**

***

## Workflow Steps

### 1. Checkout Repository

Uses `actions/checkout` to access the repository contents.

### 2. Install Dependencies

Installs:

*   **skopeo** – inspects container image metadata
*   **jq** – parses JSON output

### 3. Fetch Current Image Digest

*   Inspects the image:
        docker://mcr.microsoft.com/dotnet/sdk:10.0
*   Extracts the SHA256 digest and exposes it as a workflow output

### 4. Compare with Previous Digest

*   Reads `digest.txt` if present
*   Determines whether an update is required based on:
    *   Digest change
    *   Manual `force_update=true` input

### 5. Save Current Digest

*   Writes the current digest to `digest.txt`

### 6. Commit and Push Update (Conditional)

*   Only runs when an update is detected or forced
*   Commits the updated `digest.txt`
*   Pushes the change back to the repository

### 7. Notify via GitHub Issue (Conditional)

When an update occurs, the workflow:

*   Creates a GitHub Issue using the REST API
*   Includes:
    *   New image digest
    *   Previous image digest
    *   Reference links to Microsoft’s .NET container documentation

This notification works for both **GitHub.com** and **GitHub Enterprise Server**, using the dynamically supplied API URL.

***

## Artifacts and Outputs

*   **`digest.txt`**  
    Stores the most recently observed image digest and acts as the comparison baseline.

*   **GitHub Issue**  
    Serves as a human‑readable alert that the base image has changed.

***

## Use Cases

*   Monitoring upstream base image changes
*   Supporting supply chain security reviews
*   Triggering downstream rebuilds or manual validation
*   Auditing image updates over time via git history

***

## Future Enhancements (Optional)

*   Automatically open a Pull Request instead of a direct commit
*   Trigger dependent workflows on image update
*   Extend to monitor multiple images
*   Add Slack / Teams / email notifications
*   Add image rebuild automations
