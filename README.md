Your shell script automates the creation of an Amazon EC2 instance using the AWS CLI, while also handling error detection and AWS CLI installation. Here's a step-by-step breakdown of the script to help you write a detailed blog post.

# How to Automate EC2 Instance Creation with a Shell Script and AWS CLI
I’ll guide you through automating the creation of an EC2 instance on AWS using a bash script. We will break down the entire process of checking for AWS CLI installation, installing it if necessary, and launching an EC2 instance with specific configurations.

# 1. Overview of the Script
The script is structured into various functions to simplify the process:

1. Check if AWS CLI is installed.
2. Install AWS CLI if it isn’t already installed.
3. Wait for the instance to be in a "running" state.
4. Create the EC2 instance with specified parameters.

Let’s walk through each section to understand how the script works.

# 2. Enabling Error Handling
The script starts with:

set -euo pipefail

1. -e: Causes the script to exit immediately if any command exits with a non-zero status (i.e., if an error occurs).
2. -u: Treats unset variables as an error and immediately exits.
3. -o pipefail: Ensures that the entire pipeline fails if any part of the pipeline fails.

This combination helps detect errors early, ensuring the script doesn’t proceed if something goes wrong.

# 3. Checking if AWS CLI is Installed
The check_awscli function checks whether the AWS CLI is installed on the system.

check_awscli() {

if ! command -v aws &> /dev/null; then

echo "AWS CLI is not installed.

Please install it first." >&2 return 1

fi

}

command -v aws: Looks for the 'aws' command to determine if AWS CLI is installed.

If the CLI isn’t found, the script will print a message to 'stderr' and return '1', which signifies an error.

# 4. Installing AWS CLI
If the AWS CLI is not installed, the script runs the 'install_awscli' function to install it.

install_awscli() {

echo "Installing AWS CLI v2 on Linux..." # Download and install AWS CLI v2

curl -s "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

sudo apt-get install -y unzip &> /dev/null

unzip -q awscliv2.zip

sudo ./aws/install

aws --version

rm -rf awscliv2.zip ./aws

}

1. curl: Downloads the AWS CLI installer.
2. unzip: Unzips the downloaded installer.
3. aws/install: Runs the installation script for AWS CLI.
4. aws --version: Confirms that AWS CLI was successfully installed.

After installation, temporary files are removed to keep the system clean.

# 5. Waiting for the EC2 Instance to Start
After launching the EC2 instance, the 'wait_for_instance' function ensures that the instance reaches the "running" state before proceeding.

wait_for_instance() {

local instance_id="$1"

echo "Waiting for instance $instance_id to be in running state..."

while true; do

state=$(aws ec2 describe-instances --instance-ids "$instance_id" --query 'Reservations[0].Instances[0].State.Name' --output text)

if [[ "$state" == "running" ]]; then

echo "Instance $instance_id is now running."

break

fi

sleep 10

done

}

1. aws ec2 describe-instances: Queries the current state of the EC2 instance.
2. while true: Keeps checking the instance state every 10 seconds.
3. sleep 10: Waits for 10 seconds before checking again to avoid excessive requests.
4. This loop continues until the instance reaches the "running" state.

# 6. Creating the EC2 Instance
The 'create_ec2_instance' function uses the AWS CLI to launch an EC2 instance.

create_ec2_instance() {

local ami_id="$1"

local instance_type="$2"

local key_name="$3"

local subnet_id="$4"

local security_group_ids="$5"

local instance_name="$6"

instance_id=$(aws ec2 run-instances \

--image-id "$ami_id" \

--instance-type "$instance_type" \

--key-name "$key_name" \

--subnet-id "$subnet_id" \

--security-group-ids "$security_group_ids" \

--tag-specifications

"ResourceType=instance,Tags[{Key=Name,Value=$instance_name}]" \

--query 'Instances[0].InstanceId' \ --output text )

if [[ -z "$instance_id" ]]; then

echo "Failed to create EC2 instance." >&2

exit 1

fi

echo "Instance $instance_id created successfully."

wait_for_instance "$instance_id"

}

1. AMI ID: The Amazon Machine Image (AMI) defines the OS and software configuration of the instance.
2. Instance Type: Specifies the size of the instance (e.g., 't.micro').
3. Key Name: The SSH key used to securely access the instance.
4. Subnet ID and Security Group IDs: Define the network and security settings.
5. Tag Specification: Adds metadata to the instance, such as a name tag.
6. After creating the instance, the script uses the 'wait_for_instance' function to ensure the instance is fully running before proceeding.

# 7. Main Function
The 'main' function ties everything together.

main() {

if ! check_awscli; then

install_awscli

exit 1

fi

echo "Creating EC2 instance..."

AMI_ID="ami-085f9c64a9b75eed5"

INSTANCE_TYPE="t2.micro"

KEY_NAME="shell-scripting-for-devops"

SUBNET_ID="subnet-0e1b31b16852c6fb2"

SECURITY_GROUP_IDS="sg-04e3ad44e2064d1c1"

INSTANCE_NAME="Shell-Script-EC2-Demo"

create_ec2_instance "$AMI_ID" "$INSTANCE_TYPE" "$KEY_NAME" "$SUBNET_ID" "$SECURITY_GROUP_IDS" "$INSTANCE_NAME"

echo "EC2 instance creation completed."

}

main "$@"

# 8. Running the Script
To run the script, save it as ec2_create.sh and make it executable:

chmod +x ec2_create.sh

Then, run the script:

./ec2_create.sh

The script will check for AWS CLI, install it if necessary, and create an EC2 instance.

# Conclusion
This bash script simplifies the process of launching an EC2 instance on AWS using the AWS CLI. It automates checking for AWS CLI installation, downloading it if necessary, and handles the entire instance creation process, including waiting for the instance to fully start.

This is particularly useful for DevOps teams that want to automate cloud infrastructure management with minimal manual intervention.
