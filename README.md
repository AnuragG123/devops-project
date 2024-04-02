Here's the script without SES (Simple Email Service) functionality:

```python
import boto3
from datetime import datetime, timedelta

def delete_old_folders(bucket_names, prefix, older_than_years=8):
    s3 = boto3.client('s3')
    deleted_objects = []

    # Calculate cutoff date
    cutoff_date = datetime.now() - timedelta(days=older_than_years*365)

    for bucket_name in bucket_names:
        # List objects in the bucket with the given prefix
        response = s3.list_objects_v2(Bucket=bucket_name, Prefix=prefix)

        # Iterate through objects and delete those older than cutoff date
        if 'Contents' in response:
            for obj in response['Contents']:
                key = obj['Key']
                folder_year = key.split('=')[1][:4]  # Extract year from folder name
                folder_date = datetime.strptime(folder_year, "%Y")
                
                if folder_date < cutoff_date:
                    print(f"Deleting object: {key}")
                    s3.delete_object(Bucket=bucket_name, Key=key)
                    deleted_objects.append(key)
    
    return deleted_objects

# Example usage
if __name__ == "__main__":
    bucket_names = ['hm-add', 'hm-sub', 'fm-gh']
    prefix = 'converted/'
    deleted_objects = delete_old_folders(bucket_names, prefix)
    if deleted_objects:
        print("Deleted objects:")
        for obj in deleted_objects:
            print(obj)
    else:
        print("No objects deleted.")
```

This script deletes objects under the specified bucket names and prefix that are older than 8 years. It does not include email notification functionality. If you need email notification, you can integrate a different email service or adjust the script to suit your email notification requirements.
