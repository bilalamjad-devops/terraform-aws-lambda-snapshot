Absolutely, Bilal. I will write it for you. Since Medium articles are long, I'll write it in a professional style that is optimized for readers, recruiters, and SEO. You can copy and paste it directly into Medium.

---

# Automate AWS EBS Snapshot Cleanup Using Terraform and AWS Lambda

*Reduce AWS storage costs by automatically identifying and deleting unused EBS snapshots.*

---

## Introduction

As cloud environments grow, AWS costs can increase in unexpected ways. One common reason is forgotten Amazon EBS snapshots.

Development and operations teams frequently create snapshots before deploying a new application version, performing maintenance, or making infrastructure changes. These snapshots provide a quick rollback option if something goes wrong.

However, once the work is completed, the EC2 instance may be terminated and its EBS volume automatically deleted. The snapshot, on the other hand, remains in the AWS account. Since Amazon EBS snapshots are stored independently of the original volume, AWS continues charging for the storage they consume.

For a small environment, this may not seem like a significant issue. But in organizations that create hundreds of snapshots every month, unused snapshots slowly accumulate, increasing storage costs and making resource management more difficult.

Instead of relying on engineers to manually identify and delete stale snapshots, we can automate the process using AWS Lambda.

In this hands-on project, we'll build a simple serverless solution using **Terraform** and **AWS Lambda**. Terraform will provision the required AWS resources, while Lambda will scan the AWS account and automatically delete snapshots that are no longer associated with an existing volume.

Although this lab executes the Lambda function manually, the same solution can easily be integrated with Amazon EventBridge Scheduler to perform automatic cleanup on a daily or weekly schedule.

---

# Business Problem

Imagine a development team working on a staging environment.

Before deploying a new release, the team creates an EBS snapshot as a backup. Once testing is completed successfully, the EC2 instance is terminated.

AWS automatically deletes the attached EBS volume because **Delete on Termination** is enabled.

However, the backup snapshot is forgotten.

A month later, another deployment creates another snapshot.

Then another.

Then another.

Although these snapshots are no longer needed, AWS continues storing them, resulting in unnecessary storage costs.

In large organizations where multiple teams deploy infrastructure every day, manually tracking these snapshots becomes nearly impossible.

Automation solves this problem.

---

# Solution

In this project we will use:

* Terraform to provision the infrastructure.
* AWS Lambda to identify stale EBS snapshots.
* IAM Role and Policy to grant the required permissions.
* CloudWatch Logs to record the execution.

The workflow is simple.

1. Terraform creates the Lambda function and IAM resources.
2. We create a snapshot from an EC2 volume.
3. The EC2 instance is terminated.
4. AWS deletes the EBS volume automatically.
5. The snapshot remains in the AWS account.
6. Lambda scans all snapshots.
7. If the associated volume no longer exists, Lambda deletes the snapshot automatically.

---

# Architecture

*(Insert your architecture diagram here.)*

```
EC2 Instance
      │
      ▼
Create Snapshot
      │
      ▼
Terminate EC2
      │
      ▼
Volume Deleted
      │
      ▼
Snapshot Becomes Unused
      │
      ▼
AWS Lambda
      │
      ▼
Delete Stale Snapshot
      │
      ▼
CloudWatch Logs
```

---

# Technologies Used

* Terraform
* AWS Lambda
* Amazon EC2
* Amazon EBS
* IAM
* Amazon CloudWatch

---

# Prerequisites

Before starting this lab, make sure you have:

* An AWS account
* Terraform installed
* AWS CLI configured
* Basic understanding of AWS EC2

---

# Step 1 — Launch an EC2 Instance

To simulate a real-world environment, launch a temporary EC2 instance.

Use the following name:

```
staging-app-server-v1
```

This represents a staging application server used during development or testing.

*(Insert your EC2 screenshots here.)*

---

# Step 2 — Create an EBS Snapshot

When an EC2 instance is launched, AWS automatically creates an Amazon EBS volume for the operating system.

Navigate to **Elastic Block Store → Volumes**.

Select the root volume.

Choose **Create Snapshot**.

Give the snapshot a meaningful name such as:

```
staging-backup-before-release
```

This simulates creating a backup before deploying a new application version.

*(Insert snapshot screenshots here.)*

---

# Step 3 — Terminate the EC2 Instance

After the deployment is completed, the temporary server is no longer required.

Terminate the EC2 instance.

Because **Delete on Termination** is enabled, AWS automatically deletes the attached EBS volume.

However, notice that the snapshot still exists.

At this point, the snapshot has become an unused backup that continues consuming storage.

This is exactly the type of resource we want to clean up automatically.

*(Insert screenshots.)*

---

# Step 4 — Deploy the Infrastructure

Initialize Terraform.

```bash
terraform init
```

Review the execution plan.

```bash
terraform plan
```

Deploy the infrastructure.

```bash
terraform apply
```

Terraform creates:

* IAM Role
* IAM Policy
* AWS Lambda Function

The Lambda function contains the cleanup logic responsible for identifying unused snapshots.

*(Insert Terraform screenshots.)*

---

# Understanding the Lambda Function

The Lambda function performs the following tasks.

First, it retrieves all EBS snapshots owned by your AWS account.

Next, it verifies whether the original EBS volume still exists.

If the associated volume has already been deleted, the snapshot is considered stale.

Finally, Lambda deletes the snapshot and records the action in CloudWatch Logs.

This removes the need for engineers to manually clean up old snapshots.

*(Insert your Python code here.)*

---

# Step 5 — Verify the Deployment

Open the Lambda console.

You will notice that Terraform has created the function successfully.

The IAM role grants permission to:

* Describe snapshots
* Describe volumes
* Delete snapshots
* Write execution logs to CloudWatch

*(Insert screenshots.)*

---

# Step 6 — Test the Solution

Open the **Test** tab inside the Lambda console.

Create a test event.

For example:

```
EBSStaleSnapshotEvent
```

Click **Test**.

The Lambda function scans the account, identifies the stale snapshot, deletes it, and writes the execution details to CloudWatch Logs.

After refreshing the Snapshots page, you'll notice that the unused snapshot has been removed automatically.

*(Insert screenshots.)*

---

# Production Considerations

This project demonstrates the core automation logic using manual Lambda execution.

In a production environment, you can improve the solution by:

* Triggering Lambda automatically using Amazon EventBridge Scheduler.
* Running the cleanup every weekend or once per day.
* Applying tags to snapshots before deleting them.
* Sending Amazon SNS notifications after cleanup.
* Retaining snapshots for a defined number of days before deletion.
* Restricting IAM permissions following the principle of least privilege.

These improvements make the solution safer and more suitable for enterprise environments.

---

# Key Takeaways

In this project we learned how to:

* Provision AWS infrastructure using Terraform.
* Create a serverless cleanup solution using AWS Lambda.
* Automatically identify unused Amazon EBS snapshots.
* Reduce unnecessary AWS storage costs.
* Eliminate repetitive manual cleanup tasks.

Although this example focuses on EBS snapshots, the same automation approach can be extended to other unused AWS resources such as unattached EBS volumes, idle Elastic IP addresses, unused AMIs, or obsolete security groups.

Automation not only helps reduce cloud costs but also improves operational efficiency by allowing engineers to spend less time on repetitive maintenance tasks.

Thank you for reading!

If you found this project helpful, consider connecting with me on LinkedIn and checking out the GitHub repository for the complete source code.

### My recommendation for the GitHub repository name

I would use:

**⭐ `terraform-aws-ebs-snapshot-cleanup`**

This is clear, professional, and follows a naming convention commonly used in Terraform and open-source repositories. It also matches the project's purpose much better than `terraform-aws-lambda-snapshot`.
