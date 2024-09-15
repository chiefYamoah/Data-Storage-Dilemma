# AWS EBS and EFS Storage Solutions

This project provisions and configures **Amazon EBS** (Elastic Block Store) and **Amazon EFS** (Elastic File System) storage solutions for different use cases. The setup uses EBS for application data that persists even after instance reboots and EFS for shared data across multiple EC2 instances.

## Features

- **Amazon EBS**: A block storage volume that persists application data even after EC2 instance reboots.
- **Amazon EFS**: A shared file system that can be mounted across multiple EC2 instances, ensuring that data is accessible and editable from any instance in the setup.

## Prerequisites

Before starting, ensure the following:

- **AWS CLI** installed and configured with credentials. You can follow the setup guide [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).
- An active **AWS Account**.
- **EC2 Key Pair** for SSH access to instances.

## Setup Instructions

### Part 1: Amazon EBS Setup

#### 1. Create and Attach an EBS Volume to an EC2 Instance

1. **Launch an EC2 instance**:

   - Launch an EC2 instance in your preferred region and note its **availability zone** (e.g., `us-east-1a`).

2. **Create an EBS volume**:

   - In the AWS Management Console, navigate to **Elastic Block Store > Volumes**.
   - Create a new EBS volume in the same **availability zone** as your EC2 instance.
   - Attach the EBS volume to the EC2 instance.

3. **SSH into your EC2 instance**:
   ```bash
   ssh -i your-key.pem ec2-user@your-ec2-public-ip
   ```

#### 2. Format and Mount the EBS Volume

1. **Identify the EBS volume**:

   ```bash
   lsblk
   ```

2. **Format the EBS volume**:

   ```bash
   sudo mkfs -t ext4 /dev/xvdf  # Replace xvdf with your actual device name
   ```

3. **Mount the volume to a specific directory**:

   ```bash
   sudo mkdir /mnt/ebs-volume
   sudo mount /dev/xvdf /mnt/ebs-volume
   ```

4. **Verify the mount**:
   ```bash
   df -h
   ```

#### 3. Use EBS for Application Data

1. **Create a file on the EBS volume**:

   ```bash
   echo "This is application data stored on EBS" | sudo tee /mnt/ebs-volume/app-data.txt
   ```

2. **Persist the mount across reboots**:

   - Add an entry in `/etc/fstab` to ensure the EBS volume mounts automatically after reboot:

   ```bash
   echo "/dev/xvdf /mnt/ebs-volume ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
   ```

3. **Test persistence**:
   - Stop and start the EC2 instance.
   - Verify that the EBS volume is still mounted and the file persists:
   ```bash
   ls /mnt/ebs-volume
   cat /mnt/ebs-volume/app-data.txt
   ```

---

### Part 2: Amazon EFS Setup

#### 1. Create an Amazon EFS File System

1. **Create an EFS file system**:

   - In the AWS Management Console, go to **Amazon EFS**.
   - Create a new file system and ensure it's attached to the correct VPC.

2. **Note the EFS DNS endpoint** (e.g., `fs-xxxx.efs.us-east-1.amazonaws.com`).

#### 2. Mount the EFS File System on Multiple EC2 Instances

1. **Install NFS utilities** on both EC2 instances:

   ```bash
   sudo yum install -y nfs-utils
   ```

2. **Mount the EFS file system**:

   ```bash
   sudo mkdir /mnt/efs
   sudo mount -t nfs4 -o nfsvers=4.1 fs-xxxx.efs.us-east-1.amazonaws.com:/ /mnt/efs
   ```

3. **Verify the mount**:
   ```bash
   df -h
   ```

#### 3. Use EFS for Shared Data

1. **Create a file on one instance**:

   ```bash
   echo "Shared data on EFS" | sudo tee /mnt/efs/shared-file.txt
   ```

2. **Verify the file on the second instance**:

   ```bash
   ls /mnt/efs
   cat /mnt/efs/shared-file.txt
   ```

3. **Modify the file and verify changes**:
   ```bash
   echo "Updated data on EFS" | sudo tee -a /mnt/efs/shared-file.txt
   ```

---

### Part 3: Cleanup

#### 1. Delete the EBS Volume

1. **Unmount the EBS volume**:

   ```bash
   sudo umount /mnt/ebs-volume
   ```

2. **Delete the EBS volume**:
   - In the AWS Management Console, go to **EC2 > Volumes**.
   - Detach the volume and delete it.

#### 2. Delete the EFS File System

1. **Unmount the EFS file system** on both instances:

   ```bash
   sudo umount /mnt/efs
   ```

2. **Delete the EFS file system**:
   - In the AWS Management Console, go to **Amazon EFS**.
   - Delete the EFS file system.

#### 3. Terminate EC2 Instances

- In the AWS Management Console, go to **EC2 > Instances**.
- Select and terminate the EC2 instances.

---

## Summary of Key Steps

1. **Amazon EBS Setup**:

   - Create and attach an EBS volume.
   - Format and mount the volume.
   - Store application data and ensure persistence after instance reboot.

2. **Amazon EFS Setup**:
   - Create and mount an EFS file system.
   - Share data between multiple EC2 instances using EFS.
