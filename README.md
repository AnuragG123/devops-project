To add the region `us-west-2` to the boto3 client initialization, you can modify the code like this:

```python
import boto3

def lambda_handler(event, context):
    database_name = "your_database_name"
    result = delete_glue_database(database_name)
    return {
        'statusCode': 200 if result else 500,
        'body': "Database deleted successfully" if result else "Failed to delete database"
    }

def delete_glue_database(database_name):
    try:
        # Initialize Glue client with region specified
        glue_client = boto3.client('glue', region_name='us-west-2')
        
        # Delete the database
        response = glue_client.delete_database(Name=database_name)
        
        # Check if the operation was successful
        if response['ResponseMetadata']['HTTPStatusCode'] == 200:
            print(f"Database '{database_name}' deleted successfully.")
            return True
        else:
            print(f"Failed to delete database '{database_name}'.")
            return False
    except Exception as e:
        print(f"An error occurred: {e}")
        return False
```

This will ensure that the boto3 client operates in the `us-west-2` region.
