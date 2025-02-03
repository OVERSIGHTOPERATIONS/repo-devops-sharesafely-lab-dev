# repo-devops-sharesafely-lab-dev
Create a web application where users can securely upload files to Azure Blob Storage.
Let's break it down and get started with building your web application in Python.

1. Storage Setup
Set up Azure Blob Storage:

Create an Azure Storage account.

Create a container in the Blob Storage to store the uploaded files.

Enable data at rest encryption for added security.

2. Web Application Deployment
Develop a Web Application:

We'll use Flask as the framework for this Python web application.

Here’s a basic structure to start with:

python
from flask import Flask, request, jsonify
from azure.storage.blob import BlobServiceClient, generate_blob_sas, BlobSasPermissions
import os
import datetime

app = Flask(__name__)

# Azure Blob Storage settings
BLOB_CONNECTION_STRING = os.getenv("BLOB_CONNECTION_STRING")
BLOB_CONTAINER_NAME = "your_container_name"

blob_service_client = BlobServiceClient.from_connection_string(BLOB_CONNECTION_STRING)
container_client = blob_service_client.get_container_client(BLOB_CONTAINER_NAME)

@app.route('/upload', methods=['POST'])
def upload_file():
    file = request.files['file']
    blob_client = container_client.get_blob_client(file.filename)

    blob_client.upload_blob(file)
    sas_token = generate_blob_sas(
        account_name=blob_service_client.account_name,
        container_name=BLOB_CONTAINER_NAME,
        blob_name=file.filename,
        account_key=blob_service_client.credential.account_key,
        permission=BlobSasPermissions(read=True),
        expiry=datetime.datetime.utcnow() + datetime.timedelta(hours=1)
    )

    sas_url = f"https://{blob_service_client.account_name}.blob.core.windows.net/{BLOB_CONTAINER_NAME}/{file.filename}?{sas_token}"
    
    return jsonify({'url': sas_url})

if __name__ == '__main__':
    app.run(debug=True)
Deploy the application to Azure Web Apps:

You can use Azure CLI, GitHub Actions, or any other CI/CD pipeline to deploy your Flask application to Azure Web Apps.

3. Secure Credentials
Store sensitive credentials in Azure Key Vault:

Store your Blob Storage connection string and other sensitive information in Azure Key Vault.

Use the Azure Key Vault SDK to securely retrieve these credentials in your application.

4. Monitoring and Cleanup
Monitoring:

Use Azure Monitor to track file upload/download activities.

Cleanup:

Use Azure Functions or Logic Apps to periodically clean up expired files. Here's an example of an Azure Function in Python to delete expired blobs:

python
import os
from azure.storage.blob import BlobServiceClient
import datetime

def main(mytimer: func.TimerRequest) -> None:
    BLOB_CONNECTION_STRING = os.getenv("BLOB_CONNECTION_STRING")
    BLOB_CONTAINER_NAME = "your_container_name"
    
    blob_service_client = BlobServiceClient.from_connection_string(BLOB_CONNECTION_STRING)
    container_client = blob_service_client.get_container_client(BLOB_CONTAINER_NAME)
    
    blobs = container_client.list_blobs()
    for blob in blobs:
        if blob.creation_time < datetime.datetime.utcnow() - datetime.timedelta(days=30):  # Adjust the expiration criteria as needed
            blob_client = container_client.get_blob_client(blob.name)
            blob_client.delete_blob()

By following these steps, you'll have a secure, functional web application that allows users to upload files and generate unique, time-limited links for sharing
