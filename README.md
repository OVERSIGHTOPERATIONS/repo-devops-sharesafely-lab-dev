# repo-devops-sharesafely-lab-dev
Create a web application where users can securely upload files to Azure Blob Storage.
# Secure File Upload Web Application with Azure Blob Storage

This guide will walk you through creating a web application where users can securely upload files to Azure Blob Storage. Once uploaded, the application generates a unique, time-limited link for the user to share. This ensures that only authorized users with the link can access the uploaded file for a specified duration.

## Azure Services Used
- Azure Blob Storage
- Azure Web Apps
- Azure Key Vault

## Steps

### 1. Storage Setup
1. **Set up Azure Blob Storage**:
    - Create an Azure Storage account.
    - Create a container in the Blob Storage to store the uploaded files.
    - Enable data at rest encryption for added security.

### 2. Web Application Deployment
2. **Develop a Web Application**:
    - We'll use **Flask** as the framework for this Python web application.
    - Here’s a basic structure to start with:

```python
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
