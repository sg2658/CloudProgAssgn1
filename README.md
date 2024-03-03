An image recognition pipeline in AWS, using two parallel EC2 instances, S3, SQS, and Rekognition.
## Login to the AWS account
Here are the steps,
1. Login to your AWS account 
2. After logging into your AWS Academy account, navigate to the "Courses" section and access the "Modules" area. Locate and open the resource titled "Launch AWS Academy Learner Lab".
3. Click on Start Lab.
4. Navigate to the AWS Management Console and use the search feature to find the EC2 (Elastic Compute Cloud) service page.

## Create two EC2 instances 
5. Select the "Launch Instance" button to begin the process of deploying a new virtual machine.   // Steps for creating two EC2 instances
6. Enter the Name: cs643862-asgn1 // Here we can give instance name you want to create.
7. Select Amazon Linux 2 AMI from the drop down.
8. Select t2.micro for the instance type
9. Select the key pair. Because I have already created the key pair earlier as prerequisite. // Key Pair: Click on Key Pairs from the EC2 dashboard. 
                                                                     // Create key pair - Name:cs643862-asgn1, type: RSA, file format: .ppk (since I’m working on windows).
10. Select the security group under network section from the above steps. Because I have already created the security group as prerequisite.
      // Enter details – name: cs643862-asgn1 and relevant description. Add Inbound rules. Restrict the access for SSH port to only My IP address. Repeat the same steps for HTTP and HTTPS protocols as well. Leave the Outbound rules as is.
11. Click on Advanced details
12. Scroll to the bottom and enter the following code to the user data.
13. Userdata is a feature of EC2 instance that will run the commands when the instances bootstrap. As part of this userdata, we are installing java and also creating 
    the  credentials file to store the AWS CLI credentials.
14. Open launch template -> actions -> Launch instance from template
15. Since we need 2 instances, increase ‘Number of instances’ to 2 and click on Launch instance.
16. Check the instances and wait until the ‘status check’ changes from ‘initializing’ to ‘2/2 checks passed’.
17. Since the code must be executed on the machine, the jar files must be copied to the VMs created. For this, we are creating an S3 bucket and download the built jar 
    files in the EC2 instances.
## Creating Bucket
18. First, create an S3 bucket. Open S3 from dashboard.
19. Click on create bucket.
20. Give appropriate name to a bucket //Here the bucket named as "cs643862-asgn1".
21. No other configuration needs to be changed, scroll down and click on create bucket.
22. There will be two separate programs developed - one that utilizes recognition capabilities to identify objects in images, and another that performs text recognition.
23. The application will leverage two EC2 instances, distributing responsibilities between them. The object recognition component will operate on the first instance. 
    The second instance will host the text detection portion of the application. 
## SQS 
    In addition to EC2 and S3, we will also utilize Amazon's Simple Queue Service (SQS) in this application.
## Uploading Java programs
24. Ensure that the Java-based application logic has been bundled into executable JAR file packages for both the object detection and text detection programs. 
    We will deploy these JARs onto their respective EC2 instances to allow the applications to execute in the cloud environment. Packaging the code as JARs allows for easy deployment and execution on the Linux virtual machines.
25. We have uploaded the jar files to an S3 bucket for storage, the next step is to connect to the EC2 virtual machines using SSH. I will use the Putty tool to establish 
    the secure SSH connections to each EC2 instance. This will allow me to access the Linux VMs remotely to download and run the jar files for the object and text detection programs.Putty allows you to transfer files from your local machine over to the cloud servers.
## Putty
26. In Putty, go to Connection > SSH > Auth to specify the key pair credentials needed to access the EC2 instances.
->  Navigate to "Credentials" and browse your computer to select the .pem key file that was downloaded earlier.
->  Also under Connection, enter the SSH login details as ec2-user with the public DNS address of your EC2 instance.
->  Once the key pair and hostname are configured in Putty, we can load the connection to SSH into your cloud server.
27. I successfully connected to the first EC2 instance using SSH and Putty. The key pair and public DNS address allowed me to remotely access this cloud virtual machine.
->  I will follow the same connection steps in Putty again to open an SSH session with the second EC2 instance. I will reuse the downloaded .pem key file and get the public 
    DNS address for the second EC2 server. Entering those same credentials into a new Putty connection will let me SSH into the second cloud instance as well.
28. To confirm that Java was properly installed on the EC2 instances during setup, I will run the "java -version" command on both virtual machines after SSH successfull.
29. Now navigate to the AWS Academy Learner Lab and copy the credentials from AWS CLI section.
## Run the programs on respective EC2 instance:
30. To allow the applications to access other AWS services, we need to supply my AWS credentials to each EC2 instance.
->  Using the vi text editor over SSH, I will insert the access key, secret key, and session token into the ~/.aws/credentials file on each cloud virtual machine. 
    This will authorize the object detection and text detection programs to interact with AWS resources likes S3 storage or Rekognition.
31. After adding the AWS access credentials to the credentials file on the EC2 instances, I should validate that access is working correctly.
->  We can test this on each instance by running the "aws s3 ls" command over the SSH connection. This will use the AWS CLI to list all S3 buckets visible to those 
    inserted credentials, confirming they authorize access to my AWS account resources.
32. Both the instances listed 2 buckets on my AWS account. Now copy the Object Rekognition code to EC2-A.
## Running program on EC2-A instance (First EC2-A instance):
33. Download the object from the bucket on the EC2-A machine with command: aws s3 cp s3://cs643862-asgn1/AWSObjectRekognition.jar AWSObjectRekognition.jar
## Running program on EC2-B instance (First EC2-B instance):
34. Download the object from the bucket on the EC2-B machine with command: aws s3 cp s3://cs643862-asgn1/ AWSTextRekognition.jar AWSTextRekognition.jar
35. Both the files are download successfully. Execute the jar file on instance A first using java -jar AWSObjectRekognition.jar
36. When the Java applications run, they have been programmed to automatically create a new SQS queue called "MyQueue.fifo". This first-in-first-out queue will enable 
    the object detection and text detection programs running on separate EC2 instances to communicate with each other.
37. I can see there are currently 6 messages waiting in the "MyQueue.fifo" SQS queue that have not been processed yet.
->  To handle these pending messages, I will SSH into the second EC2 instance and run the text recognition Java application. Execute the command 
    "java -jar AWSTextRekognition.jar" over SSH to start the jar file. This should trigger the text recognition program to begin processing the 6 queued messages containing images to analyze.
## OUTPUT (COMMUNICATION OF TWO EC2s in SQS):
38. I ran the jar file with command java -jar <application.jar> first in EC2-A to recognize the image first and push it to the queue in SQS. While this EC2-A is recognizing the second image, the EC2-B will fetch the first image recognized by EC2-A present in the SQS Queue to recognize the text and prints the corresponding text of the image. 
39. Finally We will get the output as expected.




