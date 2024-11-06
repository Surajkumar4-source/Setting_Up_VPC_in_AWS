
# Setting Up a VPC (Virtual Private Cloud) in AWS with One Public and One Private Subnet.
### Step 1: Create a VPC
#### 1. Login to AWS Console: Go to AWS Management Console and log in.
#### 2. Navigate to VPC: In the AWS Management Console, search for "VPC" and click on the VPC service under "Networking & Content Delivery."
#### 3. Create VPC:
- Click on Create VPC.
- Name tag: Enter a name for your VPC (e.g., "MyVPC").
- IPv4 CIDR block: Enter a CIDR block (e.g., 10.0.0.0/16).
- IPv6 CIDR block: Leave it as "No IPv6 CIDR Block" for now.
- Tenancy: Choose Default.
- Click Create VPC.
  
### Step 2: Create Subnets
- create two subnets: one public and one private.

#### 1. Create Public Subnet:

- In the VPC dashboard, go to Subnets in the left menu.
- Click Create subnet.
- Name tag: Enter a name for the subnet (e.g., "Public-Subnet").
- VPC: Select the VPC you created in Step 1.
- Availability Zone: Choose an AZ (e.g., us-east-1a).
- IPv4 CIDR block: Enter a range (e.g., 10.0.1.0/24).
- Click Create subnet.

  
#### 2. Create Private Subnet:

- Click Create subnet again.
- Name tag: Enter a name for the subnet (e.g., "Private-Subnet").
- VPC: Select the same VPC.
- Availability Zone: Choose the same or a different AZ (e.g., us-east-1b).
- IPv4 CIDR block: Enter a range (e.g., 10.0.2.0/24).
- Click Create subnet.
  
### Step 3: Create an Internet Gateway (for Public Subnet)
#### 1. In the VPC dashboard, go to Internet Gateways.
#### 2. Click Create internet gateway.
- Name tag: Enter a name (e.g., "MyInternetGateway").
- Click Create internet gateway.
#### 3. Attach Internet Gateway to VPC:
- Select the newly created internet gateway.
- Click Actions > Attach to VPC.
- Select your VPC and click Attach.
### Step 4: Create Route Tables
*Create route tables to control traffic flow within your VPC.*

#### 1. Create Route Table for Public Subnet:

- In the VPC dashboard, go to Route Tables.
- Click Create route table.
- Name tag: Enter a name (e.g., "Public-Route-Table").
- VPC: Select your VPC.
- Click Create.
- *Edit Routes:*
   - Select the newly created route table and go to the Routes tab.
   - Click Edit routes > Add route.
   - Destination: Enter 0.0.0.0/0 (for all traffic).
   - Target: Select your Internet Gateway.
   - Click Save routes.
     
- Associate Route Table with Public Subnet:
   - Go to the Subnet Associations tab.
   - Click Edit subnet associations.
   - Select your Public Subnet and click Save.

#### 2. Create Route Table for Private Subnet:

- Click Create route table again.
- Name tag: Enter a name (e.g., "Private-Route-Table").
- VPC: Select your VPC.
- Click Create.
- Edit Routes:
    - Private subnets typically don’t have direct internet access, so leave the 
      default route.


### Step 5: Set Up NAT Gateway for Outbound Internet Access from Private Subnet

- *To allow instances in the private subnet to access the internet, you need to set up a NAT Gateway.*
  #### 1. Allocate an Elastic IP:
- In the AWS Console, go to Elastic IPs under Networking & Content Delivery.
- Click Allocate Elastic IP address and then Allocate.

#### 2. Create NAT Gateway:
- Navigate to NAT Gateways in the VPC dashboard.
- Click Create NAT Gateway.
- Subnet: Select the Public Subnet.
- Elastic IP: Choose the Elastic IP you allocated.
- Click Create NAT Gateway.

#### 3. Update Route Table for Private Subnet:
- Go back to the Private-Route-Table you created earlier.
- Click Edit routes and add the following route:
    - Destination: 0.0.0.0/0 (all traffic).
    - Target: Select the NAT Gateway.
    - Click Save routes.

 
### Step 6: Launch EC2 Instances in the Subnets
#### 1. Launch EC2 Instance in the Public Subnet:
- Go to EC2 Dashboard and click Launch Instance.
- Choose an AMI (e.g., Amazon Linux 2).
- Select an instance type (e.g., t2.micro for free tier).
- Network: Select your VPC.
- Subnet: Choose the Public Subnet.
- Auto-assign Public IP: Ensure this is enabled for internet access.
- Security Group: Configure a security group that allows inbound SSH (port 22) from your IP address.
- Key Pair: Select or create a key pair for SSH access.
- Click Launch.
  
#### 2. Launch EC2 Instance in the Private Subnet:

- Repeat the steps above, but choose the Private Subnet and disable Auto-assign Public IP.
- Configure a security group that only allows inbound SSH from the public subnet instance.
  
### Step 7: Set Up Security Groups
#### 1. Public Subnet Security Group:
- In the EC2 Dashboard, select the security group for the Public Subnet instance.
- Add an inbound rule to allow SSH (port 22) from your IP address.
  
#### 2. Private Subnet Security Group:
- Select the security group for the Private Subnet instance.
- Add an inbound rule to allow SSH (port 22) only from the public instance’s private IP address.


### Step 8: Connect to the Public EC2 Instance via SSH (using Putty).
- Open Putty (Windows) and enter the Public IP of the public EC2 instance.
- Under Connection → SSH → Auth, browse to the private key file (in .pem format) for the public EC2 instance and select it.
- Click Open to connect to the EC2 instance. You should be logged into the EC2 instance in the Public Subnet.

<br>

####  Alternate method to connect.
   -  Open a terminal or SSH client.
   - Run the SSH command using the Public IP of the Public Subnet instance:
 ```bash
  ssh -i /path/to/your-key.pem ec2-user@<public-instance-public-ip>
```


### Step 9: Access the Private Subnet EC2 Instance from the Public Subnet EC2 Instance
- SSH into the Private Subnet Instance:
- From the public instance, SSH into the Private Subnet instance using its private IP:
  - Open a terminal or SSH client.
  - Run the SSH command using the Private IP of the Private Subnet instance:
```bash
ssh -i /path/to/your-key.pem ec2-user@<private-instance-private-ip>
```

####  Alternate method to connect Private Subnet EC2 Instance from the Public Subnet EC2 Instance

- As we have downloaded .PEM key file, so first convert to .PPK file 
     - USE PUTTY GEN  SOFTWARE TO CONVERT .PEM TO .PPK file
     - Then copy the .PPK file from local machine to  Public Subnet EC2 Instance..  using WINSCP software
     - After Succesful connection, move .PEM key file to the public instance from local machine BY DRAG AND DROP
     - Now we the .PEM key file in  Public Subnet EC2 Instance.
     - Then change the .PEM key  file permission

```bash
chmod 400 PUBLIC_KEY.pem
```
   - Run the SSH command using the Private IP of the Private Subnet instance:

```bash
ssh -i  < .PEM file name >  < Private IP of the Private Subnet instance >
```
Eg. in my case:
```bash
 ubuntu@ip-172-20-1-231:~$  ls
'PUBLIC_KEY.pem'
ubuntu@ip-172-20-1-231:~$      chmod 400 PUBLIC_KEY.pem
ubuntu@ip-172-20-1-231:~$      ssh -i  "PUBLIC_KEY.pem"  172.20.2.223
```




    <br>
### This setup provides a secure, VPC configuration in AWS 



 <br>
  <br>
   <br>

# Screenshots



  

![Project Screenshot](screenshots\1.1.PNG)


![Project Screenshot](screenshots\1.2.png)




![Project Screenshot](images/your_image.png)




![Project Screenshot](images/your_image.png)



![Project Screenshot](images/your_image.png)



![Project Screenshot](images/your_image.png)



![Project Screenshot](images/your_image.png)


![Project Screenshot](images/your_image.png)


![Project Screenshot](images/your_image.png)




![Project Screenshot](images/your_image.png)

