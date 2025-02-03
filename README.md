# repo-devops-sharesafely-lab-dev
Create a web application where users can securely upload files to Azure Blob Storage.
# Secure File Upload Web Application

## Overview

This web application allows users to securely upload files to Azure Blob Storage and generates unique, time-limited links for file access.

## Azure Services Used

- Azure Blob Storage
- Azure Web Apps
- Azure Key Vault

## Steps

### 1. Storage Setup

1. Set up an Azure Blob Storage account
   - Go to the [Azure Portal](https://portal.azure.com/)
   - Create a new storage account
   - Create a container to store the uploaded files

### 2. Web Application Deployment

1. Develop a web application using Flask
   - Install Flask:
     ```bash
     pip install flask
     ```
   - Create a basic Flask application:
     ```python
     from flask import Flask, request, jsonify
     import os
     from azure.storage.blob import BlobServiceClient, generate_blob_sas, BlobSasPermissions
     from azure.keyvault.secrets import SecretClient
     from azure.identity import DefaultAzureCredential
     from datetime import datetime, timedelta

     app = Flask(__name__)

     account_url = "https://<your_storage_account>.blob.core.windows.net/"
     container_name = "<your_container_name>"
     key_vault_url = "https://<your_key_vault_name>.vault.azure.net/"
     credential = DefaultAzureCredential()
     secret_client = SecretClient(vault_url=key_vault_url, credential=credential)
     blob_service_client = BlobServiceClient(account_url=account_url, credential=credential)

     @app.route('/upload', methods=['POST'])
     def upload_file():
         file = request.files['file']
         blob_client = blob_service_client.get_blob_client(container=container_name, blob=file.filename)
         blob_client.upload_blob(file)

         sas_token = generate_blob_sas(
             account_name=blob_service_client.account_name,
             container_name=container_name,
             blob_name=file.filename,
             permission=BlobSasPermissions(read=True),
             expiry=datetime.utcnow() + timedelta(hours=1)
         )

         sas_url = f"{blob_client.url}?{sas_token}"
         return jsonify({"url": sas_url})

     if __name__ == '__main__':
         app.run(debug=True)
     ```

2. Deploy the application to Azure Web Apps
   - Use Azure CLI to deploy the application:
     ```bash
     az webapp up --name <your_app_name> --resource-group <your_resource_group> --plan <your_app_service_plan>
     ```

### 3. Secure Credentials

1. Store sensitive credentials in Azure Key Vault
   - Use DefaultAzureCredential to access these secrets securely in your application

### 4. Monitoring and Cleanup

1. Set up monitoring using Azure Monitor or Application Insights
   - Track file upload/download activities
2. Use Azure Functions or Logic Apps to clean up expired files
   - Periodically check for expired files and delete them

## Conclusion

This guide provides the steps and code required to create a secure file upload web application using Azure services. Feel free to adapt and extend the application to suit your needs.
