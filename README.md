*A brief explanation on provisioning AWS Infrastructure via Ansible and deploying Web Application.*

An Infrastructure as Code solution utilizing Ansible and the **amazon.aws** collection to provision cloud infrastructure and deploy synchronized web application stack onto AWS EC2. 

# Architecture & Workflow
## Infrastructure Provisioning (aws-infra role)
- Establishes a custom VPC network, Internet Gateway, Subnets and Route Tables with public internet routing.
- Create IAM Role and Instance Profile granting EC2 read/write access to S3 Bucket.
- Generates an EC2 Key pair, saves it locally, and launches an Ubuntu EC2 Instance with an assigned public IP.
## Asset Seeding & Synchronization (web-app role)
- Automatically seeds local application files into the target AWS S3 Bucket.
- Installs core modules ( AWS CLI with dependencies, Nginx ) on the target EC2 node.
- Synchronizes web assets securely from the S3 bucket down to */var/www/html* using an automated *--delete* sync loop.

# Directory Structure
`├── aws-infra/`
`│   ├── tasks/`
`│   │   ├── aws-ami.yml`
`│   │   ├── aws-ec2-key.yml`
`│   │   ├── aws-ec2.yml`
`│   │   ├── aws-iam.yml`
`│   │   ├── aws-rt.yml`
`│   │   ├── aws-s3.yml`
`│   │   ├── aws-sg.yml`
`│   │   ├── aws-sub.yml`
`│   │   ├── aws-vpc.yml`
`│   │   └── main.yml`
`│   └── templates/`
`│       ├── ec2-trust-policy.json.j2`
`│       └── s3-policy.json.j2`
`├── group_vars/`
`│   └── all/`
`│       ├── aws-default.yml`
`│       ├── aws-ec2.yml`
`│       ├── aws-iam.yml`
`│       ├── aws-s3.yml`
`│       ├── aws-sg.yml`
`│       └── aws-vpc.yml`
`├── web-app/`
`│   ├── files/`
`│   ├── handlers/`
`│   └── tasks/`
`│       ├── configure.yml`
`│       ├── install.yml`
`│       ├── main.yml`
`│       ├── service.yml`
`│       └── verify.yml`
`├── .gitignore`
`└── playbook.yml`

# Core Ansible Modules
## AWS Infrastructure Modules
- *amazon.aws.ec2_vpc_net* - Provisions the isolated Virtual Private Cloud ( VPC ) network.
- a*mazon.aws.ec2_vpc_subnet* - Creates the public subnet within VPC network.
- *amazon.aws.ec2_vpc_igw* - Deploys an Internet Gateway to grant outbound/inbound public internet connectivity.
- *amazon.aws.ec2_vpc_route_table* - Configures route tables to map 0.0.0.0/0 traffic directly to the Internet Gateway.
- *amazon.aws.ec2_security_group* - Defins inbound/outbound firewall rules ( SSH on 22, Web on 80/443 ).
- *amazon.aws.ec2_ami_info* - Queries AWS to dynamically fetch the latest canonical AMI ID.
- *amazon.aws.s3_bucket* - Provisions and configures the target S3 bucket.
- *amazon.aws.iam_role & amazon.aws.iam_instance_profile* - Creates least-privilege IAM roles and instance profiles so EC2 can access S3 without hardcoded API credentials.
- *amazon.aws.iam_policy* - Defines the custom JSON policy document granting read/write privileges to S3.
- *amazon.aws.ec2_key* - Generates standard SSH key pairs for instance access.
- *amazon.aws.ec2_instance* - Launches and manages the target EC2 node.
## System & Configuration Modules
- *ansible.builtin.apt* - Manages packet installation on th target node.
- *ansible.builtin.file* - Ensures proper directory exists, structure, file ownerships and permissions.
- *ansible.builtin.add_host* - Dynamically adds newly provisioned EC2 instances into an in-memory inventory group ( **launched_ec2_nodes** ) during runtime.
- *ansible.builtin.wait_for* - Polls port 22 on the launched EC2 instance untill SSH is open and ready for configuration tasks.
- *ansible.builtin.shell* - Executes local and remote CLI workflows.
- *ansible.builtin.set_fact* - Dynamically defines and sets runtime variables.
- *community.crypto.openssh_keypair* - Generates local SSH key pairs on the fly for instance access.

# Execution
To run full end-to-end deployement pipeline from the control node:
`ansible-playbook playbook.yml`
