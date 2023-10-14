## Prerequisites
- Set your IAM permissions to allow for Amazon EC2 access
- Create a [key pair](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-keypairs.html) and a [security group]([https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-sg.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-sg.html#creating-a-security-group)).
- Select an Amazon Machine Image (AMI) and note the AMI ID.

1. Create a key pair:
```bash
# Create a key pair:
aws ec2 create-key-pair --key-name <your-key-name> --query 'KeyMaterial' --output text > <key-name>.pem
# Describe/List key pair values, get details about the key.
aws ec2 describe-key-pairs --key-name <your-key-name>
```

2. Create Default VPC (*OPTIONAL*: if not already present):
```bash
aws ec2 create-default-vpc
# Describe vpcs
aws ec2 describe-vpcs
```


3.Create a security group:
- [Create Security Group.](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-sg.html#creating-a-security-group)
- [Add rules to the security group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/authorize-security-group-ingress.html).
- 
    
```bash
# if vpc-id not provided, it will create in the default VPC.
aws ec2 create-security-group --group-name my-sg --description "My security group" --vpc-id vpc-1a2b3c4d

# Add rules to the security group. Below example is to allow ssh access to linux machine. sg-0df5d1b1665071be9
aws ec2 authorize-security-group-ingress \
    --group-id sg-1234567890abcdef0 \
    --protocol tcp \
    --port 22 \
    --cidr 203.0.113.0/24
```

4. Create the Instance:
```bash
aws ec2 run-instances \
--image-id ami-0f5ee92e2d63afc18 \
--count 1 \
--instance-type t2.micro \
--key-name aws-cli-key-pair \
--security-group-ids sg-1234567890abcdef0 \
--subnet-id subnet-6e7f829e

aws ec2 run-instances --image-id ami-0f5ee92e2d63afc18 --count 1 --instance-type t2.micro --key-name aws-cli-key-pair --security-group-ids sg-0df5d1b1665071be9 --subnet-id subnet-6e7f829e
```