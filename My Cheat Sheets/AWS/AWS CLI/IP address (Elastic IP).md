To attach a public IP address (Elastic IP) to an EC2 instance using the AWS CLI, you can use the `associate-address` command. Here's how to do it:

1. First, you'll need to allocate an Elastic IP address if you haven't already. You can allocate an Elastic IP using the following command:

```bash
aws ec2 allocate-address
```

2. Take note of the Elastic IP address that is allocated; you'll need it in the next step.

3. Use the `associate-address` command to attach the Elastic IP address to your EC2 instance. Replace `your-instance-id` with the actual ID of your EC2 instance, and replace `your-elastic-ip` with the Elastic IP address you allocated in the previous step:

```bash
aws ec2 associate-address --instance-id your-instance-id --public-ip your-elastic-ip
```

Here's an example of what the command might look like with actual values:

```bash
aws ec2 associate-address --instance-id i-0123456789abcdef0 --public-ip 54.12.34.56
```

This command associates the Elastic IP address with the specified EC2 instance, making it accessible from the public internet using that IP address.