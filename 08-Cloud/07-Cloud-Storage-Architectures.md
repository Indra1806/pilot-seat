> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Cloud Storage Architectures** represent the virtualized data storage systems of the cloud. Cloud providers divide storage into three separate architectures based on access speed, capacity limits, file sharing capability, and billing rates: **Object Storage**, **Block Storage**, and **File Storage**. Selecting the correct storage engine ensures applications run quickly while minimizing hosting costs.

# Why It Exists
Historically, servers wrote all data directly to a local, physically attached hard drive. However, a single storage style cannot efficiently handle all software requirements. A database engine requires sub-millisecond speeds to write tiny table updates inside active files; a backup system needs to hold terabytes of historical logs cheaply without ever modifying them; and a cluster of web servers needs to read from the same central directory of images simultaneously. Cloud storage architectures were invented to separate storage hardware from physical servers, allowing developers to allocate cheap, slow drives for static media assets and reserve expensive, high-speed drives for databases.

# Problem It Solves
Cloud storage architectures solve the problems of storage capacity limits, server dependency, high disk costs, and file sharing limitations.

### Before Cloud Storage (Direct Attached Disk Model):
- If a server's local hard drive filled up with user photos, developers had to take the website offline, buy a larger physical drive, copy all data over, and restart the system.
- If a virtual machine crashed, all files stored on its local disk were lost or locked inside the broken VM, halting business operations.
- Multiple virtual servers could not easily read or write to the same directory of media files simultaneously, making auto-scaling difficult to configure.

### After Cloud Storage (Distributed Storage Utilities):
- Web applications write user uploads directly to Object Storage (like Amazon S3), which scales capacity infinitely and makes files accessible via web links.
- Database servers write transactional tables to Block Storage (like AWS EBS), which acts as a virtual hard drive that can be detached from a crashed server and re-attached to a healthy one in seconds.
- Multi-server clusters mount a shared File Storage directory (like Azure Files), allowing hundreds of servers to read and write to the same directories concurrently.

# Core Concepts
To design storage systems, you must understand the trade-offs of the three storage models:

1. **Object Storage:** A model that stores files as flat, individual units called "objects" (e.g. AWS S3, GCP Cloud Storage, Azure Blob Storage). Each object contains the raw file data, custom description labels (metadata), and a unique URL path identifier. You cannot edit a single byte inside an object; to make a change, you must re-upload the entire file. It is extremely cheap, has infinite capacity, and is accessible directly over HTTP.
2. **Block Storage:** A model that divides data files into raw, equal-sized blocks of bytes (e.g. AWS EBS, GCP Persistent Disk, Azure Managed Disks). It connects directly to a single virtual server as a high-performance virtual hard drive. It supports random read/write access, meaning database engines can update a single byte on a disk without touching the rest of the file. It cannot be accessed over the internet directly; it requires an attached server.
3. **File Storage:** A model that organizes data in a hierarchical tree directory structure using folders, sub-folders, and individual files (e.g. AWS EFS, GCP Filestore, Azure Files). It supports network protocols (like NFS or SMB), allowing multiple virtual servers to mount the file system and access directories simultaneously.
4. **Storage Tiering:** The practice of moving data to cheaper, slower storage tiers (e.g. S3 Glacier) as it gets older and is accessed less frequently, saving up to 80% on storage bills.

# Architecture / Components
The network pathways and connection boundaries of the three cloud storage types:

```text
  [ Web Users ]                [ Web Server VM A ]    [ Web Server VM B ]
        │                               │                      │
   (HTTP Link)                    (NFS Network)          (NFS Network)
        │                               ▼                      ▼
        ▼                      ┌────────────────────────────────────────┐
  ┌───────────┐                │ Shared File Storage (e.g. AWS EFS)     │
  │ Object    │                │  /var/www/uploads/ (Hierarchical)      │
  │ Storage   │                └────────────────────────────────────────┘
  │ (S3/Blob) │
  │  Files    │                [ Database VM ]
  │  Metadata │                     │
  └───────────┘                (Fiber Channel)
                                    ▼
                               ┌────────────────────────────────────────┐
                               │ Block Storage volume (e.g. AWS EBS)    │
                               │  [Block 1] [Block 2] [Block 3] [Raw]    │
                               └────────────────────────────────────────┘
```

# Workflow
How an application utilizes different storage architectures during a typical user transaction:

```text
Step 1: User uploads a new profile picture. The web app receives the file and sends it to Object Storage (S3).
                             ↓
Step 2: S3 stores the image, generates a public URL, and returns the path link to the web app.
                             ↓
Step 3: The web app connects to the database VM to save the user's profile URL.
                             ↓
Step 4: The database VM writes the URL text block directly into its table files stored on Block Storage (EBS).
                             ↓
Step 5: The database writes transactional log files to a shared network directory on File Storage (EFS).
                             ↓
Step 6: Other worker VMs in the cluster read the logs from EFS to update search index records.
```

# Real World Examples
Think of cloud storage models as **valet storage, personal desk drawers, and hallway filing cabinets**.
- **Object Storage (Valet storage warehouse):** You pack your camping gear into a plastic bin, label it, and hand it to a valet service. They store it somewhere in a giant warehouse. They hand you a claim ticket. When you need the gear, you show the ticket and they bring the bin. If you want to change one flashlight inside, you cannot edit it in the warehouse; you must retrieve the entire bin, change the flashlight, and return the bin.
- **Block Storage (A personal desk drawer):** A drawer physically bolted to your personal office desk (Virtual Machine). You open it, grab a notebook, write a single word, and drop it back in. It is extremely fast, but only you can open it. If your desk breaks, you must unscrew the drawer and bolt it onto a new desk to get your notebook back.
- **File Storage (Shared hallway filing cabinet):** A central filing cabinet located in the building hallway. It contains labeled hanging folders. Multiple coworkers walk up, open drawers, browse directories, read files, and write new reports simultaneously. It is slower to search than your desk drawer, but everyone shares access.

# Implementation
Here is how developers interact with storage systems. Example A shows how to upload a file to Object Storage using the AWS SDK, and Example B shows the commands a systems engineer runs to mount a shared File Storage system on a Linux virtual machine:

### Example A: Uploading to Object Storage (Node.js SDK)
```javascript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";

// 1. Initialize the S3 storage client
const s3 = new S3Client({ region: "us-east-1" });

export const uploadFile = async (fileName, fileContent) => {
    const uploadParams = {
        Bucket: "my-app-uploads-bucket",
        Key: `images/${fileName}`, // Object path key
        Body: fileContent, // Raw file bytes
        ContentType: "image/jpeg"
    };

    try {
        // 2. Submit the put object request to the S3 API
        const data = await s3.send(new PutObjectCommand(uploadParams));
        console.log("File uploaded successfully, Object version ID:", data.VersionId);
        return `https://my-app-uploads-bucket.s3.amazonaws.com/images/${fileName}`;
    } catch (err) {
        console.error("Error uploading file to Object Storage:", err);
    }
};
```

### Example B: Mounting Network File Storage (Linux CLI)
```bash
# 1. Install the standard NFS client utilities
sudo apt-get update
sudo apt-get install -y nfs-common

# 2. Create the directory path where the shared files will live
sudo mkdir -p /var/www/shared-media

# 3. Mount the remote File Storage system using its DNS endpoint
# We use NFS version 4 and specify the cloud storage target address
sudo mount -t nfs4 \
  -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
  fs-08e1e721a302.efs.us-east-1.amazonaws.com:/ \
  /var/www/shared-media

# 4. Confirm the mount is active and check disk space
df -h | grep shared-media
```

# Best Practices
- **Never Host Databases on Object Storage:** Relational database systems (like Postgres or MySQL) rely on random byte-level writes. Object storage (S3/Blob) only supports replacing files entirely, which will crash database engines. Use Block Storage for database installations.
- **Enable Object Versioning and MFA Delete:** Protect files in Object Storage from accidental deletion or ransomware attacks by enabling versioning (which keeps copies of previous file updates) and MFA Delete (requiring multi-factor authentication to delete files).
- **Configure Automated Lifecycle Policies:** Do not store old log files on expensive hot storage. Write rules to automatically demote files to archive storage (e.g. S3 Glacier) after 90 days, and delete them entirely after 1 year.

# Industry Standards
Cloud storage achieves high durability through **Multi-Region Replication**. Cloud providers write object storage files across multiple separate datacenters simultaneously. For example, AWS S3 Standard provides "eleven 9s" of durability ($99.999999999\%$). This guarantees that if you store 10,000,000 files in the cloud, you might lose a single file once every 10,000 years, safeguarding data against datacenter disasters.

# Common Mistakes
- **Exposing Private Buckets Globally:** Leaving an Object Storage bucket configured as "publicly readable" to make it easy for the app to load images. Attackers scan cloud IP ranges looking for public buckets, leading to data leaks of customer records. Keep buckets private and use **Presigned URLs** to grant temporary access.
- **Writing User Files to Ephemeral Disk:** Saving user documents to the local disk of a virtual machine instance. Local VM disks are often temporary (ephemeral); if the server crashes or scales down, all files are lost.
- **Ignoring Block Volume IOPS Limits:** Attaching a low-tier block storage disk to a high-traffic database. Block storage performance is capped by IOPS (Input/Output Operations Per Second). Databases exceeding these limits will freeze up, delaying user requests.

# Security & Performance Considerations
- **Storage Encryption Keys:** Encrypt all storage volumes using Key Management Services (KMS). This ensures that if a physical hard drive is discarded or stolen from a cloud datacenter, the raw data cannot be read.
- **Block Storage Tier Customization:** Choose SSD-backed volumes (like AWS gp3 or Azure Premium SSD) for database active files, and cost-effective HDD-backed volumes (like AWS st1) for streaming media files or long-term system logs.

# Related Technologies
- **MinIO:** An open-source, high-performance object storage server that implements the Amazon S3 API, used to test object storage systems locally.
- **NFS (Network File System):** The protocol standard used to mount and share files across Linux systems.
- **SMB (Server Message Block):** The Microsoft networking protocol used to share files between Windows clients and cloud file storage.

# Summary

## What We Learned
- Object storage (S3/Blob) holds flat files, is infinitely scalable, and is accessed via web URLs.
- Block storage (EBS/Managed Disks) serves as virtual hard drives attached to single servers for high-speed, random byte-level writes.
- File storage (EFS/Files) provides hierarchical directory folders that can be shared across multiple servers concurrently.
- Lifecycle rules automatically transition old files to archive tiers to optimize cloud spending.

## Key Takeaways
- Use Object Storage for all static assets (images, scripts, documents) to optimize costs and enable easy file retrieval.
- Restrict Block Storage to database engines and operating system files that require sub-millisecond read/write speeds.
- Secure all Object Storage buckets by blocking public access and using temporary Presigned URLs for authenticated file delivery.

# Keywords
- Cloud Storage
- Object Storage
- Block Storage
- File Storage
- Storage Durability
- Storage Tiering
- Amazon EBS
- Amazon S3
- Amazon EFS
- Lifecycle Policy

# Glossary

| Term | Meaning |
|---|---|
| Object Storage | Flat storage system where data is stored as individual objects with metadata and a web key identifier. |
| Block Storage | High-speed storage where files are split into raw sectors (blocks) and connected as virtual server drives. |
| File Storage | Shared storage that organizes data into hierarchical folders and files using network sharing protocols. |
| Durability | The probability that stored data remains intact and readable without becoming corrupted or lost over time. |
| Ephemeral Disk | Temporary storage attached to a virtual server that is wiped clean when the server restarts or terminates. |
| Presigned URL | A temporary web link generated by an owner granting access to download a private file for a limited time. |

## Next Recommended Chapters
- 08-Cloud-Database-Solutions.md
