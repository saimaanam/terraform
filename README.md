
# ACS730 Assignment1 – Ansible Static Web Deployment  
Configuration Management of a Heterogeneous EC2 Fleet

---

1. Purpose of this README
This document lists everything you must have in place _before_ you run the Ansible playbook and walks you through applying the configuration to the _pre‑provisioned_ EC2 instances.


---

2. Prerequisites
Control Node
Cloud9 Environment (or any host with Python ≥ 3.9): Cloud9 comes pre-configured with AWS credentials and VPC access, making it ideal for this assignment.

Ansible ≥ 2.15: Required for the dynamic inventory plugin and newer amazon.aws modules.

Python packages boto3 and botocore: These libraries let Ansible interact with AWS services like EC2.

Terraform ≥ 1.7 (optional): Use this only if you need to provision or destroy the EC2 infrastructure.

Credentials
IAM permissions: Your Cloud9 IAM role or user must have ec2:Describe* permissions so Ansible can discover EC2 instances.

SSH Key Pair: You must have the private key (~/.ssh/week6-dev.pem by default) corresponding to the key pair attached to your EC2 instances.

Target EC2 Instances
Tagging Requirements:

Role=web — used by Ansible to dynamically discover relevant instances.

OS=Ubuntu or OS=Amazon — used to group hosts for OS-specific configuration.

Security Group: Must allow inbound SSH (port 22) from the Cloud9 instance's CIDR.

Python 3 Installed: Ansible requires Python on the target instance. Ubuntu has it by default; Amazon Linux 2 usually does, but older AMIs may not.

---

2.1 Installing Ansible & boto3 on Cloud9

```bash
# Inside Cloud9 terminal
python3 -m pip install --user --upgrade ansible boto3 botocore
```

---

2.2 Required edits to `~/.bashrc`  (or `~/.bash_profile`)
Add the following lines once, then `source ~/.bashrc`:

```bash
# Alias for Terraform (handy during grading)
alias tf='terraform'
```

---

2.3 Ansible configuration (`ansible.cfg`)

```ini
[defaults]
inventory          = inventory/aws_ec2.yaml   # dynamic inventory plugin
private_key_file   = ~/.ssh/mykey.pem         # adjust if you use a different key
host_key_checking  = False                    # avoid host‑key prompts
retry_files_enabled= False
forks              = 10                       # speed‑up for small fleets
timeout            = 20
```
---

3. Project Layout

```
.
├── ansible.cfg
├── playbook.yml
├── inventory/
│   └── aws_ec2.yaml
├── group_vars/
│   ├── os_amazon_linux/
│   │   └── main.yml
│   └── os_ubuntu/
│       └── main.yml
└── roles/
    └── web/
        ├── tasks/
        │   └── main.yml
```


4. Running the Configuration

1. Clone or unzip this repository in Cloud9.  
2. `cd terraform/dev && tf init && tf apply` – creates the EC2 instances.  
3. Back in the project root, run the playbook:

   ```bash
   ansible-playbook -i inventory/aws_ec2.yaml playbook.yml
   ```

   - Ansible auto‑discovers every instance with `Role=web`.  
   - Ubuntu hosts receive `apache2`; Amazon hosts receive `httpd`.  
   - Each server gets an `index.html` that prints its hostname and your name.

4. **Verify:

   ```bash
   ansible -i inventory/aws_ec2.yaml all -a "curl -s http://localhost | head -2"
   ```

---

## 5. Troubleshooting Checklist

If Ansible reports “UNREACHABLE: Permission denied (publickey),” it typically means the SSH key is incorrect or missing. 
Double-check that the private key path in your ansible.cfg file matches the key used when launching the instances. 
If Ansible finds “0 hosts matched,” your EC2 instances may not be properly tagged with Role=web; add the tag using the AWS Console or CLI and re-run the playbook. 
If Ansible fails because “python3: command not found,” your AMI might be outdated; install Python 3 manually (sudo yum install -y python3) or use a newer Amazon Linux or Ubuntu image. 
Finally, if your web browser displays the Apache test page instead of your custom site, the playbook may not have copied index.html correctly; 
verify write permissions to /var/www/html and re-run the playbook to ensure your template was deployed.

---

6. Cleaning Up

If you used Terraform:

```bash
cd terraform/dev
tf destroy
```

Otherwise simply terminate the EC2 instances in the AWS Console.

---

7. Credits

Created by Saima Anam Syed for ACS 730 – Assignment 3 (June 2025).
