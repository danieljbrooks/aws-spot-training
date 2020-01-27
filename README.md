# aws-spot-training - Under Construction! - (Living document)

Tutorial on training models with AWS spot instances. 

Steps:

Choose a region. I choose US West Oregon (Note that some GPU instances in limited regions.)

1. Create "brooks-model" bucket in US West (Oregon) [has p3 instances]
  - Enable versioning.
  - Enable AES-256 encryption.
  - Block all public access.
  - Create bucket
  
2. EC2
  - Services -> region -> us-west-2
  - Launch template -> Create launch template
  - Choose template name - brooks-ec2
  - Choose template description - "Launch EC2 spot instances for training."
  - AMI -> "Deep Learning AMI (Ubuntu 18.04) Version 26.0"
  - Instance type -> p3.2xlarge (Available in US-West-2)
  - Key pair (currently using brooks-ec2)
  - Network settings (EC2-Classic)
  - Availability zone - "Don't include in launch template"
  - Security groups - New window, EC2 -> Security Groups
    - Name -> brooks-ec2
    - Description -> Security group for training instances. 
    - Default VPC (public)
    - Inbound: Type: SSH, Port range 22, source Anywhere (Note: restrict to my IP later)
    - Outbound: All traffic, any port
    - Description: EC2 for model training.
  - Refresh security groups and add to template.
  - Storage volumes (encrypt->yes, leave key blank, size->100GB, delete on termination)
  - Instance tags: Name -> spot-training (instances & volumes)
  - Advanced details -> request spot -> True
  - IAM
    - New window -> iam
    - Create policy -> s3 (all actions, s3 bucket) [use s3FullAccess]
    - Name "brooks-spot-bucket-access"
    - Description: full_access_to_brooks-model
  - IAM Role -> EC2 -> brooks-spot-bucket-access
  - Tags: Name: spot-train
  - Role name: brooks-spot-train
  - Create role
  - Go back to EC2 Launch template, add role. 
  - User data will become useful later. 
  
  - Click on launch template -> can modify.  
  
  - Current able to ssh into instance

3. For S3 bucket - allow 

To provide restricted access to an s3 bucket, try a policy of the form:

'''{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::yourbucketname/*"
        }
    ]
}'''

