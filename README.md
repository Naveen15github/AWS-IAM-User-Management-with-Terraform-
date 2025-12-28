# AWS-IAM-User-Management-with-Terraform

## 📌 Project Overview

This project demonstrates how to **automate AWS IAM user, group, and access management using Terraform**.
The entire IAM setup is driven by a CSV file, making it scalable, repeatable, and ideal for real-world enterprise environments.

The solution dynamically creates IAM users,configures console access and applies metadata using tags — all with infrastructure as code principles.

This project is inspired by real-world identity management use cases and closely aligns with how organizations manage access at scale.

---
## Architecture Diagram
![Architecture](screenshots/01-terraform-init.png)

# 📸 Proof of Work – AWS IAM User Management with Terraform

This section documents the complete implementation using screenshots captured during execution.

---

## 🔹 1. Terraform Initialization

**Description:**
Terraform backend initialized using an S3 remote backend with versioning enabled.

![Terraform Init](screenshots/01-terraform-init.png)

---

## 🔹 2. Terraform Plan Output

**Description:**
Preview of infrastructure changes before applying them.


![Terraform Plan](screenshots/02-terraform-plan.png)

---

## 🔹 3. Terraform Apply Execution

**Description:**
Successful creation of IAM users, groups, and related resources.

![Terraform Apply](screenshots/03-terraform-apply.png)

---

## 🔹 4. IAM Users Created

**Description:**
Verification of IAM users created dynamically from the CSV file.
![IAM Users](screenshots/04-iam-users.png)

## 🚀 Key Features

* Automated creation of **multiple IAM users**
* Dynamic **group assignment based on department**
* **CSV-driven architecture** (single source of truth)
* IAM **console access** enabled for users
* Fully **idempotent** Terraform configuration
* Centralized **S3 backend** for Terraform state
* Clean, modular, and production-ready structure

---

## 🧱 What This Project Creates

### ✅ AWS Resources

* **26 IAM Users**
* **3 IAM Groups**

  * Education
  * Managers
  * Engineers
* **Group memberships** automatically mapped
* **User tags** for department, job title, and display name

---

## 📁 Project Structure

```
aws-iam-user-management/
│
├── backend.tf          # Remote S3 backend configuration
├── provider.tf         # AWS provider setup
├── versions.tf         # Terraform & provider version constraints
├── main.tf             # User creation & CSV decoding logic
├── groups.tf           # Group creation & membership assignment
├── users.csv           # Source of truth for all IAM users
├── outputs.tf          # Output definitions
├── DEMO_GUIDE.md       # Step-by-step demo explanation
└── README.md           # Project documentation
```

---

## 🛠️ Prerequisites

Before starting, ensure you have:

* AWS CLI configured (`aws configure`)
* Terraform v1.0 or later
* AWS account with IAM permissions
* S3 bucket for remote state storage

---

## ⚙️ Setup Instructions

### 1️⃣ Create S3 Backend for Terraform State

```bash
aws s3 mb s3://my-terraform-state-bucket-piyushsachdeva --region us-east-1

aws s3api put-bucket-versioning \
  --bucket my-terraform-state-bucket-piyushsachdeva \
  --versioning-configuration Status=Enabled
```

---

### 2️⃣ Initialize Terraform

```bash
terraform init
```

---

### 3️⃣ Review the Execution Plan

```bash
terraform plan
```

---

### 4️⃣ Apply the Configuration

```bash
terraform apply -auto-approve
```

---

### 5️⃣ Verify in AWS Console

Navigate to:

* **IAM → Users** (verify all users)
* **IAM → User Groups**
* **IAM → Security Credentials**

---

## 📄 CSV-Driven User Management

### Example `users.csv`

```csv
first_name,last_name,department,job_title
Michael,Scott,Education,Regional Manager
Dwight,Schrute,Sales,Assistant to the Regional Manager
Jim,Halpert,Sales,Sales Representative
Pam,Beesly,Reception,Receptionist
Ryan,Howard,Temps,Temp
```

This CSV acts as the **single source of truth** for user creation.

---

## 🔍 How It Works (Technical Breakdown)

### Step 1: Read CSV File

```hcl
locals {
  users = csvdecode(file("users.csv"))
}
```

---

### Step 2: Create IAM Users Dynamically

```hcl
resource "aws_iam_user" "users" {
  for_each = { for user in local.users : user.first_name => user }

  name = lower("${substr(each.value.first_name, 0, 1)}${each.value.last_name}")
  path = "/users/"

  tags = {
    DisplayName = "${each.value.first_name} ${each.value.last_name}"
    Department  = each.value.department
    JobTitle    = each.value.job_title
  }
}
```

---

### Step 3: Enable Console Access

```hcl
resource "aws_iam_user_login_profile" "users" {
  for_each = aws_iam_user.users

  user                    = each.value.name
  password_reset_required = true
}
```

---

### Step 4: Create IAM Groups

```hcl
resource "aws_iam_group" "education" {
  name = "Education"
}

resource "aws_iam_group" "managers" {
  name = "Managers"
}

resource "aws_iam_group" "engineers" {
  name = "Engineers"
}
```

---

### Step 5: Assign Users to Groups Dynamically

```hcl
resource "aws_iam_group_membership" "education_members" {
  name  = "education-members"
  group = aws_iam_group.education.name

  users = [
    for user in aws_iam_user.users :
    user.name if user.tags.Department == "Education"
  ]
}
```

---

## 📤 Outputs

```bash
terraform output account_id
terraform output user_names
terraform output user_passwords
```

> ⚠️ Passwords are sensitive and should be handled securely.

---

## 👥 Sample Users Created

| Username | Full Name      | Department | Job Title                         |
| -------- | -------------- | ---------- | --------------------------------- |
| mscott   | Michael Scott  | Education  | Regional Manager                  |
| dschrute | Dwight Schrute | Sales      | Assistant to the Regional Manager |
| jhalpert | Jim Halpert    | Sales      | Sales Representative              |
| pbeesly  | Pam Beesly     | Reception  | Receptionist                      |
| rhoward  | Ryan Howard    | Temps      | Temp                              |

---

## 🔐 Security Best Practices

* Enforce password reset on first login
* Use IAM groups instead of attaching policies to users
* Enable MFA for all users
* Do not commit `terraform.tfstate`
* Enable CloudTrail for audit logging
* Prefer AWS SSO for production environments

---

## 🧹 Cleanup

To delete all created resources:

```bash
terraform destroy
```

---

## 📈 Future Enhancements

* Attach IAM policies to groups
* Integrate AWS SSO
* Add MFA enforcement
* Automate onboarding from HR systems
* Add CI/CD validation for Terraform code

---

## 📚 References

* Terraform AWS Provider Documentation
* AWS IAM Best Practices
* Terraform Functions Documentation

