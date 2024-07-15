To integrate email notifications into your Lambda function for sending alerts when folders are deleted, you'll need to use the AWS SES (Simple Email Service) to send emails. Here's how you can modify your Lambda function to include email notifications:

```python
import boto3
from datetime import datetime
import re
import logging

# Initialize boto3 clients
s3_client = boto3.client('s3')
ses_client = boto3.client('ses')

# Define constants
BUCKET_NAMES = ['anuragwarangal', 'rameshbucket']

# Define paths for each bucket
PATHS = {
    'anuragwarangal': [
        'converted/loss/on533/region_number=',
        'converted/loss/onomc/offc=',
        'converted/premium/oneko/offc=',
        'converted/premium/onfepm/offc='
    ],
    'rameshbucket': [
        'converted/loss/oness/office=',
        'converted/loss/onest/office=',
        'converted/premium/onfip/',
        'converted/premium/onfld/office='
    ]
}

# Define suffixes for office and region_number
OFFC_SUFFIXES = [str(i).zfill(2) for i in range(1, 29)]  # 01 to 28
REGION_SUFFIXES = [str(i).zfill(2) for i in range(1, 29)]  # 01 to 28

# Set up logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def get_folders_to_delete(bucket_name, prefix, date_pattern, date_format):
    folders_to_delete = set()
    for suffix in OFFC_SUFFIXES if 'office=' in prefix else REGION_SUFFIXES:
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
                    folder_date = datetime.strptime(date_str, date_format)
                    logger.info(f"Found folder date: {folder_date}")
                    if (datetime.now() - folder_date).days > 8 * 365:  # Older than 8 years
                        logger.info(f"Folder {sub_prefix} is older than 8 years and will be deleted")
                        folders_to_delete.add(sub_prefix)
                else:
                    # If no date match found, drill down one more level
                    sub_response = s3_client.list_objects_v2(Bucket=bucket_name, Prefix=sub_prefix, Delimiter='/')
                    if 'CommonPrefixes' in sub_response:
                        for sub_common_prefix in sub_response['CommonPrefixes']:
                            sub_sub_prefix = sub_common_prefix['Prefix']
                            logger.info(f"Checking sub-sub-prefix: {sub_sub_prefix}")
                            match = re.search(date_pattern, sub_sub_prefix)
                            if match:
                                date_str = match.group(1)
                                folder_date = datetime.strptime(date_str, date_format)
                                logger.info(f"Found folder date: {folder_date}")
                                if (datetime.now() - folder_date).days > 8 * 365:  # Older than 8 years
                                    logger.info(f"Folder {sub_sub_prefix} is older than 8 years and will be deleted")
                                    folders_to_delete.add(sub_sub_prefix)
                            else:
                                logger.info(f"No date match found in sub-sub-prefix: {sub_sub_prefix}")
                    else:
                        logger.info(f"No sub-sub-prefixes found in sub-prefix: {sub_prefix}")
        else:
            logger.info(f"No sub-prefixes found in prefix: {full_prefix}")
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
    
    # Send email notification
    send_email_notification(folders)

def send_email_notification(folders_deleted):
    subject = f"S3 Folder Deletion Notification"
    body = f"The following folders were deleted:\n\n"
    for folder in folders_deleted:
        body += f"{folder}\n"
    
    try:
        response = ses_client.send_email(
            Destination={
                'ToAddresses': [
                    'anurag09@gmail.com',
                    'rahul11@gmail.com'
                ]
            },
            Message={
                'Body': {
                    'Text': {
                        'Charset': 'UTF-8',
                        'Data': body,
                    },
                },
                'Subject': {
                    'Charset': 'UTF-8',
                    'Data': subject,
                },
            },
            Source='your_verified_email@example.com'  # Replace with your verified SES email
        )
        logger.info("Email notification sent")
    except Exception as e:
        logger.error(f"Failed to send email notification: {str(e)}")

def lambda_handler(event, context):
    logger.info("Lambda function started")

    # Define patterns and formats for each bucket
    patterns_formats = {
        'anuragwarangal': [
            (r'RPT_DT=(\d{8})/', '%Y%m%d'),  # For RPT_DT folders
            (r'YR_MO=(\d{6})/', '%Y%m')      # For YR_MO folders
        ],
        'rameshbucket': [
            (r'RPT_DT=(\d{8})/', '%Y%m%d'),  # For RPT_DT folders
            (r'YR_MO=(\d{6})/', '%Y%m'),     # For YR_MO folders
            (r'FILE_DATE=(\d{6})/', '%Y%m')  # For FILE_DATE folders
        ]
    }

    # Combine all folders to delete
    folders_to_delete = []
    for bucket_name in BUCKET_NAMES:
        for prefix in PATHS[bucket_name]:
            for pattern, date_format in patterns_formats[bucket_name]:
                folders_to_delete += get_folders_to_delete(bucket_name, prefix, pattern, date_format)

    if folders_to_delete:
        for bucket_name in BUCKET_NAMES:
            delete_folders(bucket_name, folders_to_delete)
            logger.info(f"Deleted folders in {bucket_name}: {folders_to_delete}")
    else:
        logger.info("No folders to delete")

    return {
        'statusCode': 200,
        'body': 'Lambda function executed successfully.'
    }
```

### Explanation:
1. **Integration with SES**: Added `boto3.client('ses')` to initialize the SES client for sending emails.
2. **Email Notification Functionality**: Introduced `send_email_notification()` function to construct and send email notifications to specified recipients (`anurag09@gmail.com` and `rahul11@gmail.com`) after folders are deleted.
3. **SES Configuration**: Replace `'your_verified_email@example.com'` with your verified email address in AWS SES console. SES requires email addresses to be verified before you can send emails from them.
4. **Usage in `delete_folders()`**: Modified `delete_folders()` function to call `send_email_notification()` after deleting folders to notify recipients.

Ensure that your Lambda function has the necessary IAM permissions to interact with S3 (read and delete objects) and SES (send email). Also, make sure your SES account is properly configured with the necessary permissions and verified email addresses. Adjust the SES configuration and email content as per your specific requirements.
