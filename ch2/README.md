# Deploy EC2 Instance with Bash Script 

### 1. Prepare the `.env` File
	•	In your project directory, create a file named `.env`:

```
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_DEFAULT_REGION=us-east-2

```

	•	Never commit this file to version control.
### 2. Copy the User Data Script
	•	Copy the app’s user data script into the deployment directory.

```
cp ch1/ec2-user-data-script/user-data.sh ch2/bash/

```

### 3. Write the Bash Deployment Script
	•	In `ch2/bash/`, create a file called `deploy-ec2-instance.sh` with these contents:

```#!/usr/bin/env bash
set -e

set -a
source .env
set +a

export AWS_DEFAULT_REGION="${AWS_DEFAULT_REGION:-us-east-2}"

user_data=$(cat user-data.sh)

security_group_id=$(aws ec2 create-security-group \
  --group-name "sample-app" \
  --description "Allow HTTP traffic into the sample app" \
  --output text \
  --query GroupId)

aws ec2 authorize-security-group-ingress \
  --group-id "$security_group_id" \
  --protocol tcp \
  --port 80 \
  --cidr "0.0.0.0/0" > /dev/null

image_id=$(aws ec2 describe-images \
  --owners amazon \
  --filters 'Name=name,Values=al2023-ami-2023.*-x86_64' \
  --query 'reverse(sort_by(Images, &CreationDate))[:1] | [0].ImageId' \
  --output text)

instance_id=$(aws ec2 run-instances \
  --image-id "$image_id" \
  --instance-type "t2.micro" \
  --security-group-ids "$security_group_id" \
  --user-data "$user_data" \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=sample-app}]' \
  --output text \
  --query Instances[0].InstanceId)

public_ip=$(aws ec2 describe-instances \
  --instance-ids "$instance_id" \
  --output text \
  --query 'Reservations[*].Instances[*].PublicIpAddress')

echo "Instance ID = $instance_id"
echo "Security Group ID = $security_group_id"
echo "Public IP = $public_ip"

```

### 4. Make the Script Executable
	•	Change directory and set execute permissions:

```cd ch2/bash
chmod u+x deploy-ec2-instance.sh

```

### 5. Install AWS CLI
	•	If not installed, follow AWS documentation for your OS.

### 6. Run the Script
	•	Just execute:

```./deploy-ec2-instance.sh

```

	•	Output includes the new Instance ID, Security Group ID, and Public IP address.
### 7. Test Your EC2 Instance
	•	Open a browser to `http://<Public IP>` using the address given at the end of the script.
	•	You should see your app’s “Hello, World!” page once the instance boots up.
