# Shell Script Task for Cost Optimization Using S3  

## Objective  
Optimize the cost of storing and managing logs by replacing ELK stack storage with Amazon S3 for backup and long-term storage. This approach reduces storage costs and automates the log backup process using Jenkins, shell scripting, and AWS S3 lifecycle management.  

---

## Proposed Solution  

1. **Transition from ELK to S3 for Logs Storage**  
   - Instead of using ELK for log storage and backup, utilize Amazon S3, which offers significantly lower costs.  
   - ELK can still be used for analysis, but logs no longer need to be retained in ELK beyond the analysis stage.  

2. **Automation of Log Backup to S3**  
   - Automate the daily log backup process using a shell script integrated with **Jenkins** and scheduled via **cron jobs**.  
   - Store logs in an S3 bucket designated for backup and archival purposes.  

3. **Leverage S3 Lifecycle Policies**  
   - Define lifecycle policies to manage logs efficiently:  
     - Move infrequently accessed logs to **S3 Glacier** or **S3 Deep Archive** for cost-effective storage.  
     - Automatically delete logs after a predefined retention period (e.g., 90 or 180 days).  

4. **Daily Log Backup Process**  
   - Create a shell script to:  
     - Compress log files.  
     - Upload the compressed files to the specified S3 bucket.  
   - Configure a cron job to execute this script every night.  

5. **Jenkins Integration**  
   - Use Jenkins to trigger the shell script for log upload and monitoring.  
   - Replace email notifications with an automated log archival system, eliminating unnecessary communication overhead.  

---

## Benefits  

- **Cost Optimization**: Reduced costs by transitioning from ELK storage to S3 for logs backup.  
- **Automation**: Eliminates manual tasks using Jenkins and shell scripting.  
- **Efficient Storage Management**: S3 lifecycle rules ensure cost-effective retention of logs in Glacier or Deep Archive.  
- **Scalability**: S3 can handle log data growth without additional infrastructure changes.  

---

## AWS EC2 Instance

- Go to AWS Console
- Instances(running)
- Launch instances

<img width="994" alt="Screenshot 2023-02-01 at 12 37 45 PM" src="https://user-images.githubusercontent.com/43399466/215974891-196abfe9-ace0-407b-abd2-adcffe218e3f.png">

### Install Jenkins.

Pre-Requisites:
 - Java (JDK)

### Run the below commands to install Java and Jenkins

Install Java

```
sudo apt update
sudo apt install openjdk-17-jre
```

Verify Java is Installed

```
java -version
```

Now, you can proceed with installing Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to Jenkins using the below URL:

http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">

### Click on Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">

Wait for the Jenkins to Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]

<img width="990" alt="Screenshot 2023-02-01 at 11 02 09 AM" src="https://user-images.githubusercontent.com/43399466/215959757-403246c8-e739-4103-9265-6bdab418013e.png">

Jenkins Installation is Successful. You can now starting using the Jenkins 

<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">

# Step 1: Create an S3 Bucket in AWS Console
1. Log in to the AWS Management Console.
2. Navigate to **S3** from the AWS Services menu.
3. Click on **Create Bucket**.
4. Enter a bucket name and select the desired region.
5. Configure additional settings as needed (e.g., versioning, encryption).
6. Click **Create Bucket**.

# Step 2: Install AWS CLI on your Linux Server

## Download the AWS CLI Installer
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

## Install `unzip` if not already installed
```bash
sudo apt install unzip -y
```

## Unzip the AWS CLI Installer
```bash
unzip awscliv2.zip
```

## Install AWS CLI
```bash
sudo ./aws/install
```

## Verify the AWS CLI Installation
```bash
aws --version
```

# Step 3: Configure AWS CLI

## Set Up AWS Credentials
```bash
aws configure
```
- Provide the following when prompted:
  - **AWS Access Key ID**: Your AWS access key.
  - **AWS Secret Access Key**: Your AWS secret key.
  - **Default region name**: The region you want to use (e.g., `us-east-1`).
  - **Default output format**: Typically `json`, `text`, or `table`.

## Verify Credentials Configuration
```bash
aws sts get-caller-identity
```
This command confirms if your credentials are configured correctly by returning account and user details.

# Step 4: Write a Shell Script with `vim` to Automate AWS Resource Interaction

## Open `vim` Editor
```bash
vim s3upload.sh
```
## Save and Exit
1. Press `ESC`.
2. Type `:wq` and press `ENTER`.

## Make the Script Executable
```bash
chmod 777 s3upload.sh
```

## Run the Script
```bash
./s3upload.sh
```

# Step 5: Verify Uploaded Logs in S3
1. After running the script, any logs or files uploaded will be stored in the specified S3 bucket.
2. Open the AWS Management Console.
3. Navigate to **S3** and select your bucket.
4. Check for the uploaded files or logs.

