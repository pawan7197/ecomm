🚀 Full Deployment Guide: E-Commerce App on AWS

1. 🏗️ Setup the EC2 Infrastructure

Step 1.1: Create a Security Group

A Security Group acts like a firewall. It controls who can access your EC2 instances.

Open AWS Console → go to EC2 > Security Groups

Click Create Security Group

Fill in:

Name: Ecom

Description: Security group for your e-commerce app EC2 instances.

Add Inbound Rules:

SSH (port 22): So you can connect to the EC2 via terminal.

HTTP (port 80): So people can access the website.

📸 Add Image: Screenshot of Security Group Inbound Rule Configuration Here
![Image](https://github.com/user-attachments/assets/bebadcde-0a90-4a0f-a74c-1ec85a98abc0)


2. 🚚 Deploy the E-Commerce App on EC2 Instances

Step 2.1: Launch EC2 Instances

You will launch two Linux EC2 instances in two different zones to make the app highly available.

Go to EC2 > Launch Instance

Launch 2 instances with:

AMI: Amazon Linux

Instance Type: t2.micro

Security Group: Ecom

Key Pair: Create new → Devops

AZ 1: us-east-1a (EC2-1A)

AZ 2: us-east-1b (EC2-1B)

📸 Add Image: EC2 Launch Summary Page Here



Step 2.2: Install App on EC2

SSH into both instances using your terminal:

ssh -i Devops.pem ec2-user@<public-ip>

Then run this setup:

sudo yum update -y
sudo yum install git httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo git clone https://github.com/pawan7197/ecomm.gitsudo rm -rf /var/www/html/*
sudo cp -r ecomm/* /var/www/html


Step 2.3: Access the Website

Go to your browser and type:

http://<EC2-1A-public-ip>

http://<EC2-1B-public-ip>

You’ll now see the E-Commerce app live.

📸 Add Image: E-Commerce App Web Page from Browser
![Image](https://github.com/user-attachments/assets/255c2203-f442-4a93-a61b-3ab7b4bae683)
![Image](https://github.com/user-attachments/assets/10d04e46-8271-4baf-bef0-eae3b1b3f0c8)


3. ⚖️ Configure Load Balancer (ALB)

Step 3.1: Create Target Group

Go to EC2 > Target Groups

Click Create Target Group

Use:

Name: Ecom-TG

Protocol: HTTP

Port: 80

Target Type: Instances

Register both EC2s to this group.

Click Create

📸 Add Image: Target Group Creation and Registration Screenshot
![Image](https://github.com/user-attachments/assets/44b05f85-9b5b-4286-9760-565376e62928)
![Image](https://github.com/user-attachments/assets/ae075b62-c4ae-48e5-8c2b-9fa5762493b7)

Step 3.2: Create Application Load Balancer

Go to EC2 > Load Balancers > Create Load Balancer

Choose Application Load Balancer

Set:

Name: Ecom-ALB

Scheme: Internet-facing

Listeners: HTTP on port 80

AZs: us-east-1a, us-east-1b

Security Group: Ecom-SG

Target Group: Ecom-TG

📸 Add Image: Load Balancer Configuration Summary

![Image](https://github.com/user-attachments/assets/f5302443-dfb9-4b94-a34e-5d10aad8ba5a)

![Image](https://github.com/user-attachments/assets/0d1b807e-e23e-4d65-b082-19e9734fd0f0)
Step 3.3: Test Load Balancer

Once ALB is active, open:

http://<load-balancer-dns-name>

This should open your app using ALB.

📸 Add Image: App Accessed via ALB URL

4. 📈 Set Up Auto Scaling

Step 4.1: Create an AMI

Go to EC2 > Instances

Select one EC2 instance

Choose Actions > Create Image

Name: Ecom-AMI

Uncheck reboot, click Create Image

Wait until AMI status = available

📸 Add Image: AMI Creation and Status Available
![Image](https://github.com/user-attachments/assets/6fbdc81c-0028-4f0b-868e-0fa8fd54640b)

![Image](https://github.com/user-attachments/assets/e47a6a2e-d525-4200-a108-9348b6019bc3)

Step 4.2: Create Launch Template

Go to EC2 > Launch Templates

Click Create launch template

Set:

Template Name: Ecom-LTEMP

AMI: Ecom-AMI

Instance Type: t2.micro

Key Pair: joy

Security Group: Ecom-SG

Click Create template

📸 Add Image: Launch Template Summary

Step 4.3: Create Auto Scaling Group

Go to EC2 > Auto Scaling Groups

Click Create Auto Scaling group

Set:

Name: Ecom-ASG

Launch Template: Ecom-LTEMP

AZs: us-east-1a, us-east-1b

Attach Load Balancer:

Select ALB created earlier

Choose Target Group: Ecom-TG

Configure Group Size:

Min: 2 instances

Desired: 2

Max: 4

Set Scaling Policy:

Type: Target Tracking

Metric: CPU Utilization

Target: 70%

Optional: Set email alerts using SNS.

Click Create Auto Scaling Group

📸 Add Image: Auto Scaling Group Configuration Summary
![Image](https://github.com/user-attachments/assets/f354ae08-122f-446b-9753-0c3db1efb165)

5. 🥐 Validate Deployment

Go to EC2 > Instances → You should see Auto Scaling created instances.

Go to Target Groups > Targets → Check instance health.

Open Load Balancer DNS → Your app should work fine.

📸 Add Image: Auto Scaling Instances and Healthy Target Status
![Image](https://github.com/user-attachments/assets/aba26d46-0d6c-449a-a917-46e1e088b32b)

6. 🔍 Test Auto Scaling with Load (Stress Test)

Step 6.1: Install Stress Tool

SSH into any instance in the Auto Scaling group and run:

sudo yum install stress -y
stress --cpu 1 --timeout 10000

This increases CPU usage, triggering Auto Scaling.

📸 Add Image: Stress Tool Running and CloudWatch Graph

Step 6.2: Monitor in CloudWatch

Go to CloudWatch > Alarms

Observe if new EC2s are added when CPU is high.

Check:

EC2 console for new instances

Target Group → All should show Healthy

SNS emails for scale-up alerts (if enabled)

📸 Add Image: CloudWatch Alarm and SNS Notification
![Image](https://github.com/user-attachments/assets/c26dda3f-ebd6-4890-a75e-e560cb0b8377)

🎉 You have successfully deployed and tested a scalable E-Commerce app using EC2, ALB, and Auto Scaling on AWS!

