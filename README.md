```markdown
# AWS Distributed Storage & Web Hosting Laboratory

This repository documents the implementation of a distributed storage architecture using **Amazon S3** as a backend for an **Apache2** web server hosted on **Amazon EC2**. This setup replaces traditional local disk management (RAID/GlusterFS) with a scalable, cloud-native object storage solution.

## 🏗 System Architecture

The architecture decouples the web application from the physical storage, ensuring high durability and easy scalability.

* **Instance:** Amazon EC2 (Ubuntu 24.04 LTS)
* **Backend Storage:** Amazon S3 (Object Storage)
* **File System Interface:** `s3fs-fuse`
* **Web Server:** Apache2 (HTTP/HTTPS)
* **SSL/TLS:** Let's Encrypt (Certbot)



## 🛠 Prerequisites

* An AWS Account.
* An EC2 instance (Ubuntu 24.04) with an **IAM Instance Profile** granting `AmazonS3FullAccess`.
* A domain name (DuckDNS or custom .com) pointed to the EC2 Public IP.
* Inbound Rules for Port 22 (SSH), Port 80 (HTTP), and Port 443 (HTTPS) enabled in the AWS Security Group.



## 🚀 Installation & Configuration

### 1. Storage Integration (S3 Mount)
Install the `s3fs` utility and mount the bucket to a local directory. Using IAM roles avoids storing hardcoded credentials.

```bash
# Update system and install s3fs
sudo apt update && sudo apt install s3fs -y

# Create the mount point
sudo mkdir -p /data

# Mount the bucket (Replace <your-bucket-name> and <IAM-Role-Name>)
sudo s3fs <your-bucket-name> /data -o iam_role=<IAM-Role-Name> -o allow_other -o url=[https://s3.amazonaws.com](https://s3.amazonaws.com)
```

### 2. Web Server Setup (Apache2)
Install Apache and configure it to serve files directly from the S3 mount using a symbolic link.

```bash
# Install Apache2
sudo apt install apache2 -y

# Link S3 storage to the web root
sudo rm -rf /var/www/html
sudo ln -s /data /var/www/html
```

### 3. Application Deployment (Modern Modal)
Create a modern index file using the HTML5 `<dialog>` element to validate the connection.

```bash
sudo tee /data/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; background: #f0f2f5; }
        button { padding: 12px 24px; background: #007bff; color: white; border: none; border-radius: 6px; cursor: pointer; }
        dialog { border: none; border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.1); padding: 2rem; }
        dialog::backdrop { background: rgba(0,0,0,0.5); }
    </style>
</head>
<body>
    <h1>S3 Cloud Storage Active ✅</h1>
    <button onclick="document.getElementById('m7').showModal()">Validate Connection</button>
    <dialog id="m7">
        <p>This content is served from S3 via an EC2 mount point.</p>
        <button onclick="document.getElementById('m7').close()">Close</button>
    </dialog>
</body>
</html>
EOF
```

### 4. Security Configuration (SSL/TLS)
Secure the application with a free SSL certificate from Let's Encrypt using Certbot.

```bash
# Install Certbot for Apache
sudo apt install certbot python3-certbot-apache -y

# Obtain and install the certificate
# Replace <your-domain.com> with your actual domain
sudo certbot --apache -d <your-domain.com>
```



## ✅ Validation Procedures

### File Persistence Test
1.  Upload a file directly to the S3 Bucket via the AWS Console.
2.  Verify the file appears instantly in `/var/www/html` on the EC2 instance.
3.  Access the file via `https://your-domain.com/filename`.

### Redundancy Verification
Simulate a "drive failure" by unmounting the storage:
```bash
sudo umount /data
```
Refresh the browser to confirm the site is offline. Re-run the mount command to confirm instant recovery of all data from the S3 cloud backend.
