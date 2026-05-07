# AWS Distributed Cloud Storage & Web Hosting Lab

This project demonstrates the transition from traditional local disk management (RAID/GlusterFS) to a cloud-native distributed storage architecture. By mounting an **Amazon S3 bucket** as a local file system on an **EC2 instance**, we create a highly durable and scalable web hosting environment.

## 🏗 Architecture Overview

The system architecture replaces physical disk arrays with a virtualized cloud backend:

* **Compute:** Amazon EC2 (Ubuntu 24.04 LTS).
* **Storage:** Amazon S3 (Object storage mounted as block storage).
* **FUSE Layer:** `s3fs-fuse` for mounting S3 buckets to the Linux file system.
* **Web Server:** Apache2 serving content via symbolic links.
* **Security:** HTTPS/TLS via Let's Encrypt (Certbot) and AWS IAM roles for secure resource access.



## 🚀 Key Features

* **Distributed Storage:** Decouples web content from the compute instance, allowing for infinite storage scaling.
* **Modern UI:** Implements native HTML5 `<dialog>` elements for modern, library-free modals.
* **Automated Security:** Full HTTPS implementation with automated certificate renewal.
* **Resiliency:** Data persists in S3 even if the EC2 instance is terminated or the local mount is detached.

## 🛠 Tech Stack

* **Cloud:** AWS (EC2, S3, IAM, Route 53/DuckDNS)
* **Server:** Apache HTTP Server
* **Security:** Certbot / Let's Encrypt
* **Storage Interface:** s3fs-fuse
* **Frontend:** HTML5, CSS3 (Modern Modals)

## 📖 Setup Summary

### 1. Storage Integration
The S3 bucket is mounted to `/data` using an IAM Instance Profile, ensuring no hardcoded credentials exist on the server.
```bash
sudo s3fs your-bucket-name /data -o iam_role=auto -o allow_other -o url=[https://s3.amazonaws.com](https://s3.amazonaws.com)
```

### 2. Web Server Configuration
Apache is configured to "consume" the S3 data through a symbolic link, making the cloud storage the source of truth for the web root.
```bash
sudo rm -rf /var/www/html
sudo ln -s /data /var/www/html
```

### 3. Modern Frontend Implementation
The project utilizes modern HTML components for user interaction:
```html
<button onclick="document.getElementById('m7-modal').showModal()">Validate Setup</button>

<dialog id="m7-modal">
    <h3>Validation Success ✅</h3>
    <p>Served from S3 via EC2 mount.</p>
    <button onclick="document.getElementById('m7-modal').close()">Close</button>
</dialog>
```

## ✅ Validation & Testing

To validate the distributed nature of the storage:
1.  **File Persistence:** Upload a file to the S3 bucket; it appears instantly in the `/var/www/html` directory of the EC2 instance.
2.  **Mount Recovery:** Unmounting the `/data` folder simulates a drive failure; remounting restores the website without data loss.
3.  **SSL Validation:** Secured via Port 443 with an automated redirect from Port 80.
