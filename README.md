To extend your Lambda function to send emails using Amazon SES when folders are deleted, you'll need to perform the following steps:

1. Configure SES for sending emails.
2. Update your Lambda function to use SES.
3. Add necessary IAM permissions to allow Lambda to send emails via SES.

### Step 1: Configure SES for Sending Emails

Make sure your email addresses are verified in SES (Amazon SES requires email addresses to be verified for sending and receiving emails).

### Step 2: Update Lambda Function to Use SES

Update your Lambda function to send emails using SES.

```python
import boto3
from datetime import datetime, timedelta
import re
import logging
import os

# Initialize boto3 clients
s3_client = boto3.client('s3')
ses_client = boto3.client('ses')

# Define constants
BUCKET_NAMES = ['anuragwarangal', 'rameshbuckethm']
SES_SENDER_EMAIL = os.environ['SES_SENDER_EMAIL']
SES_RECIPIENT_EMAILS = os.environ['SES_RECIPIENT_EMAILS'].split(',')

# Define suffixes for office, offc, and region_number
OFFICE_SUFFIXES = [str(i).zfill(2) for i in range(1, 29)]  # 01 to 28
OFFC_SUFFIXES = [str(i).zfill(2) for i in range(1, 29)]  # 01 to 28
REGION_SUFFIXES = [str(i).zfill(2) for i in range(1, 29)]  # 01 to 28

# Set up logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def get_folders_to_delete(bucket_name, prefix, date_pattern, date_format):
    folders_to_delete = set()
    suffixes = OFFICE_SUFFIXES if 'OFFICE=' in prefix else OFFC_SUFFIXES if 'OFFC=' in prefix else REGION_SUFFIXES
    for suffix in suffixes:
        full_prefix = f"{prefix}{suffix}/"
        logger.info(f"Listing objects in bucket {bucket_name} with prefix {full_prefix}")
        response = s3_client.list_objects_v2(Bucket=bucket_name, Prefix=full_prefix, Delimiter='/')
        if 'CommonPrefixes' in response:
            for common_prefix in response['CommonPrefixes']:
                sub_prefix = common_prefix['Prefix']
                logger.info(f"Checking sub-prefix: {sub_prefix}")
                match = re.search(date_pattern, sub_prefix)
                if match:
                    date_str = match.group(1)
                    try:
                        if date_format == '%Y%m':  # For YR_MO
                            year = date_str[:4]
                            month = date_str[4:6]
                            folder_date = datetime(int(year), int(month), 1)
                        elif date_format == '%Y%m%d':  # For RPT_DT and FILE_DATE
                            year = date_str[:4]
                            month = date_str[4:6]
                            day = date_str[6:8]
                            folder_date = datetime(int(year), int(month), int(day))
                        logger.info(f"Found folder date: {folder_date}")
                        if (datetime.now() - folder_date).days > 8 * 365:  # Older than 8 years
                            logger.info(f"Folder {sub_prefix} is older than 8 years and will be deleted")
                            folders_to_delete.add(sub_prefix)
                    except ValueError:
                        logger.warning(f"Ignoring invalid date format in folder: {sub_prefix}")
    return list(folders_to_delete)

def delete_folders(bucket_name, folders):
    for folder in folders:
        logger.info(f"Deleting contents of folder: {folder}")
        response = s3_client.list_objects_v2(Bucket=bucket_name, Prefix=folder)
        if 'Contents' in response:
            for obj in response['Contents']:
                logger.info(f"Deleting object: {obj['Key']}")
                s3_client.delete_object(Bucket=bucket_name, Key=obj['Key'])
        logger.info(f"Deleting folder: {folder}")
        s3_client.delete_object(Bucket=bucket_name, Key=folder)

def send_email(folders_deleted):
    subject = "S3 Folder Deletion Notification"
    body_text = f"The following folders were deleted:\n" + "\n".join(folders_deleted)
    body_html = f"<html><body><h3>S3 Folder Deletion Notification</h3><p>The following folders were deleted:</p><ul>" + \
                "".join(f"<li>{folder}</li>" for folder in folders_deleted) + "</ul></body></html>"

    response = ses_client.send_email(
        Source=SES_SENDER_EMAIL,
        Destination={
            'ToAddresses': SES_RECIPIENT_EMAILS
        },
        Message={
            'Subject': {
                'Data': subject
            },
            'Body': {
                'Text': {
                    'Data': body_text
                },
                'Html': {
                    'Data': body_html
                }
            }
        }
    )

    logger.info(f"Email sent! Message ID: {response['MessageId']}")

def lambda_handler(event, context):
    logger.info("Lambda function started")

    # Define patterns and formats for each bucket
    patterns_formats = {
        'anuragwarangal': [
            (r'YR_MO=(\d{6})/', '%Y%m'),     # For YR_MO folders in on533 and onomc
            (r'RPT_DT=(\d{8})/', '%Y%m%d')   # For RPT_DT folders in oneko and onfpm
        ],
        'rameshbuckethm': [
            (r'YR_MO=(\d{6})/', '%Y%m'),     # For YR_MO folders in oness and onest
            (r'RPT_DT=(\d{8})/', '%Y%m%d'),  # For RPT_DT folders in onfip
            (r'FILE_DATE=(\d{8})/', '%Y%m%d')  # For FILE_DATE folders in onfld
        ]
    }

    # Combine all folders to delete
    folders_to_delete = []
    
    for bucket_name in BUCKET_NAMES:
        if bucket_name == 'anuragwarangal':
            paths = {
                'converted/loss/on533/region_number=': [
                    (r'YR_MO=(\d{6})/', '%Y%m')
                ],
                'converted/loss/onomc/OFFC=': [
                    (r'YR_MO=(\d{6})/', '%Y%m')
                ],
                'converted/premium/oneko/OFFC=': [
                    (r'RPT_DT=(\d{8})/', '%Y%m%d')
                ],
                'converted/premium/onfpm/OFFC=': [
                    (r'RPT_DT=(\d{8})/', '%Y%m%d')
                ]
            }
        elif bucket_name == 'rameshbuckethm':
            paths = {
                'converted/loss/oness/OFFICE=': [
                    (r'YR_MO=(\d{6})/', '%Y%m')
                ],
                'converted/loss/onest/OFFICE=': [
                    (r'YR_MO=(\d{6})/', '%Y%m')
                ],
                'converted/premium/onfip/RPT_DT=': [
                    (r'RPT_DT=(\d{8})/', '%Y%m%d')
                ],
                'converted/premium/onfld/OFFICE=': [
                    (r'FILE_DATE=(\d{8})/', '%Y%m%d')
                ]
            }
        
        for prefix, patterns in paths.items():
            for pattern, date_format in patterns_formats[bucket_name]:
                folders_to_delete += get_folders_to_delete(bucket_name, prefix, pattern, date_format)

    if folders_to_delete:
        for bucket_name in BUCKET_NAMES:
            delete_folders(bucket_name, folders_to_delete)
            logger.info(f"Deleted folders in {bucket_name}: {folders_to_delete}")
        send_email(folders_to_delete)
    else:
        logger.info("No folders to delete")

    return {
        'statusCode': 200,
        'body': 'Lambda function executed successfully.'
    }
```

### Step 3: Update Terraform Configuration

Update your Terraform configuration to include the necessary environment variables and IAM policy for SES.

#### `variables.tf`

Add new variables for SES.

```hcl
variable "ses_sender_email" {
  description = "The email address to send notifications from"
  type        = string
  default     = "your-sender-email@example.com"
}

variable "ses_recipient_emails" {
  description = "A comma-separated list of email addresses to send notifications to"
  type        = string
  default     = "anurag@gmail.com,rahul@gmail.com"
}
```

#### `main.tf`

Update the environment variables for your Lambda function and add the IAM policy for SES.

```hcl
# Add SES permissions to the Lambda policy
resource "aws_iam_policy" "lambda_policy" {
  name        = "lambda-policy"
  description = "Policy for Lambda function"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "*"
      },
      {
        Effect   = "Allow"
        Action   = [
          "s3:ListBucket",
          "s3:GetObject",
          "s3:DeleteObject"
        ]
        Resource = "*" 

 # Replace with your specific S3 bucket ARNs if needed
      },
      {
        Effect   = "Allow"
        Action   = "ses:SendEmail"
        Resource = "*"
      }
    ]
  })
}

# Update Lambda environment variables
resource "aws_lambda_function" "s3_cleanup" {
  ...
  environment {
    variables = {
      SES_SENDER_EMAIL      = var.ses_sender_email
      SES_RECIPIENT_EMAILS  = var.ses_recipient_emails
    }
  }
  ...
}
```

### Apply Terraform Configuration

Initialize and apply the Terraform configuration.

```bash
terraform init
terraform apply
```

This setup will ensure that your Lambda function can send emails using SES when folders are deleted. Make sure to replace placeholder values like `"your-sender-email@example.com"` with actual email addresses and adjust policies as necessary.
