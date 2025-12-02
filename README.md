Runbook – Validating Managed VM Configuration Changes (PR Review Guide)
1. Purpose

This runbook provides the steps and validation checks required when reviewing configuration changes for Managed VMs deployed through the Landing Zone Spokes repository.
It ensures consistent, compliant, and error-free deployments across environments (DEV/NONPROD/PROD).

2. Scope

This applies to all pull requests that modify or introduce new:

<workload>.prod.json

<workload>.nonprod.json

<workload>.dev.json

…within the Landing Zone Spokes repository under workload folders such as ES, Informatica, etc.

3. Validation Steps
3.1 Confirm File Structure

Open the changed .json file and ensure the following top-level structure exists:

environment

workloadShortName

subscription_id

connectivity

vm_configurationset

Minimal object hierarchy must be present and correctly formatted.

3.2 Validate Subscription and Spoke Mapping

These values must match an existing Landing Zone spoke.

Checklist:

subscription_id corresponds to the correct environment

spoke_canvas.resource_group_name exists

spoke_canvas.virtual_network_name exists

spoke_canvas.subnet_name exists

➡️ If unsure, compare to another working workload configuration.

3.3 Validate Connectivity Block

These values rarely change.

Check:

vwan_hub_id matches standard VWAN hub for the environment

bastion_id matches other workloads in same environment

Action: Compare with an existing PROD file.

3.4 Validate VM Configuration Set
3.4.1 General Mandatory Fields

Confirm presence of:

display_name

location

vm_size

instances

os.type and os.sku

data_disks (even if empty)

os_disk_storage_type

maintenance_window (if custom maintenance is enabled)

backup_solutions

3.4.2 Business Logic Checks
| Parameter              | Expected Behavior                             |
| ---------------------- | --------------------------------------------- |
| `public_ip`            | Must be **FALSE** unless explicitly requested |
| `aad_login`            | Should be **TRUE** for Managed VMs            |
| `allow_update_reboots` | TRUE for PROD unless stated otherwise         |
| `rpo_hours`            | Validate according to DR requirement          |
| `owners`               | Must contain valid corporate emails           |
| `useOldSSHKey`         | Relevant only for Linux workloads             |


3.4.3 Maintenance Window

If present:

recur_every uses correct format (e.g., 1Week Monday)

start_time is in UTC

duration is typically 02:00

3.4.4 Backup Settings

Ensure compliance:

"backup_solutions": ["RecoveryServicesVault"]

3.5 Tag Validation

Check presence of required tags:

"tags": {
  "PatchPriority": "business-critical"
}


Ensure no typos or missing mandatory tags.

3.6 Cross-File Comparison

Compare the updated file with a known working configuration from the same environment.

Tools recommended:

VS Code → Compare Selected

Notepad++ Compare plugin (if preferred)

3.7 Syntax Validation

Before approving:

JSON valid (correct commas, braces, quotes)

GitHub PR UI typically highlights syntax errors, but manual check still recommended

Use any JSON linter/validator if required

3.8 Pipeline Test (Optional but Recommended)

Once the PR is correct and approved:

Step 1 – Open the Managed VM Pipeline

Navigate to the pipeline responsible for Managed VM deployments.

Choose the branch:

Default: Always select main branch

Exception: If the change is specific to another branch, select that branch accordingly

Step 2 – Select Pipeline Stages

When the pipeline UI loads, all stages may be selected by default.

Unselect all stages first

Select only the stage that corresponds to the workload/environment being changed.

Examples:

If the file changed is iam.prod.json → select the IAM-PROD stage

If the file changed is es.nonprod.json → select the ES-NONPROD stage

📝 The stage name typically matches the workload short name + environment.

Step 3 – Run the Pipeline

Click Run

Validate the following:

Validation stage passes

Deployment plan is created successfully

No missing variables or module resolution errors

If validation succeeds, the configuration is safe to deploy.

5. Common Concerns to Watch For

| Issue                              | Why It Matters                       |
| ---------------------------------- | ------------------------------------ |
| Wrong `subscription_id`            | Deployment fails entirely            |
| Misspelled VNet or subnet          | VM fails to attach to network        |
| Missing maintenance window         | Pipeline validation fails            |
| Incorrect VWAN/bastion IDs         | Causes routing or access issues      |
| `public_ip = true` unintentionally | Violates security policy             |
| Incorrect VM size                  | May exceed quota or cost constraints |
