[&larr; AWS CloudFormation](README.md)

### AWS CloudFormation: assignment

#### Task 1

Develop a CloudFormation template called "networking" that includes the following AWS networking components:

- VPC (with "EnableDnsSupport" and "EnableDnsHostnames" options enabled);
- non-default route table;
- two subnets in different availability zones associated with the route table (with "MapPublicIpOnLaunch" option enabled);
- Internet gateway attached to the VPC;
- default route in the route table pointing to the Internet gateway.

The template must have the following input parameters:

- CIDR block for VPC (a default value must be pre-defined);
- CIDR block for subnet A (a default value must be pre-defined);
- CIDR block for subnet B (a default value must be pre-defined).

The template must have the following outputs:

- VPC ID;
- subnet A ID;
- subnet B ID.

#### Task 2

Develop a CloudFormation template called "security_groups" that includes the following AWS networking components:

- "Admin" (allowing all incoming connections from your IP address and the IP address of Karaganda EPAM office);
- "ELB" (allowing incoming HTTP connections from anywhere);
- "EC2" (allowing incoming HTTP connections from members of "ELB" security group);
- "RDS" (allowing incoming MySQL connections from members of "EC2" security group).

The template must have the following input parameter:

- VPC ID.

The template must have the following outputs:

- security group IDs.

#### Task 3

Develop a CloudFormation template called "database" that includes the following RDS database and related components:

- DB subnet group that includes subnets created in a previous task;
- DB parameter group of "MySQL5.7" family with "slow_query_log" set according to template parameter value;
- RDS instance using the created subnet / parameter groups, with 8 GB of General Purpose SSD storage, and having "Admin" and "RDS" security groups attached. "Multi-AZ" option should NOT be enabled.

The template must have the following input parameters:

- IDs of subnets to place RDS instance in;
- IDs of "Admin" and "RDS" security groups;
- RDS instance class (a list of allowed values must be pre-defined, "db.t2.micro" being the default);
- MySQL "slow_query_log" setting (allowed values: 0 or 1);
- database username;
- database password (this value must not be visible when entered during stack creation).

The template must have the following outputs:

- RDS endpoint address.

#### Task 4

Develop a CloudFormation template called "web" that includes the following web stack components:

- internet-facing classic Elastic Load Balancer;
- Launch Configuration (EC2 instance must be based on Amazon Linux 2 AMI and have a User Data script that installs Apache/php and creates "index.php" script that displays hostname, tries to connect to the database, and indicates whether it succeeded);
- Auto Scaling group linked with the Elastic Load Balancer.

The template must have the following input parameters:

- subnet IDs;
- security group IDs for ELB;
- security group IDs for EC2 instances;
- EC2 instance class (a list of allowed values must be pre-defined, "t2.micro" being the default);
- minimum ASG instance count (a default value must be pre-defined);
- maximum ASG instance count (a default value must be pre-defined);
- EC2 key pair name;
- RDS endpoint address;
- database username;
- database password  (this value must not be visible when entered during stack creation).

The template must have a mapping to choose AMI image ID based on the AWS region.

The template must have the following outputs:

- ELB domain name.

#### Task 5

Create a key pair for EC2 using AWS Management Console or AWS CLI.

Develop a CloudFormation template called "main" to build the entire environment using the four templates created earlier. 
The template must have the following input parameters and pass them to the underlying templates:

- CIDR block for VPC (a default value must be pre-defined);
- CIDR block for subnet A (a default value must be pre-defined);
- CIDR block for subnet B (a default value must be pre-defined);
- EC2 instance class (a list of allowed values must be pre-defined, "t2.micro" being the default);
- minimum ASG instance count (a default value must be pre-defined);
- maximum ASG instance count (a default value must be pre-defined);
- EC2 key pair name;
- RDS instance class (a list of allowed values must be pre-defined, "db.t2.micro" being the default);
- MySQL "slow_query_log" setting (allowed values: 0 or 1);
- database username;
- database password (this value must not be visible when entered during stack creation).

An existing EC2 key pair must be used during stack creation.

The template must have the following outputs:

- ELB domain name.

#### Task 6

Verify that you can successfully build CloudFormation stacks using the developed templates, and upload the templates to a publicly-accessible S3 bucket.

#### Expected results

1.	Your public S3 bucket must contain five CloudFormation templates:

- networking.json (or networking.yaml);
- security_groups.json (or security_groups.yaml);
- database.json (or database.yaml);
- web.json (or web.yaml);
- main.json (or main.yaml).

2.	When creating a CloudFormation stack using the main template, the administrator should be presented with a set of parameters to customize the environment (described in the corresponding task).

3.	As a result of "main" stack creation, four additional related stacks must be created, and the following AWS components must be provisioned:

- VPC (including an IGW, a routing table, and two subnets) with CIDR blocks specified by the administrator;
- security groups limiting access to ELB, EC2 and RDS instances;
- RDS MySQL instance with instance class, "slow_query_log" setting, database name and credentials specified by the administrator;
- Launch Configuration and Auto Scaling group with AMI image automatically set according to AWS region, and EC2 key name, instance size, and count specified by the administrator;
- Classic Elastic Load Balancer with HTTP listener.

4.	The ELB's HTTP port must be publicly accessible. EC2 and RDS instances must have no publicly accessible ports, but accept incoming connections from the IP address of EPAM's Karaganda office.

5.	Opening the ELB's domain name in browser must result in a dynamic web page being rendered. The page must indicate the web server’s hostname, and a successful MySQL connection between the web instance and the RDS database server.