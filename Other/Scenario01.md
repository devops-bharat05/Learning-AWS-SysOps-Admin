## Scenario01: Ssuppose if I have running windows aws ec2 running in private subnet with IP 198.162.0.1 and taken AMI backup of ec2 instance. After pushing windows patching manually application running on ec2 stop working or server went down and not coming up then how I can restore the same server back from AMI with same IP?

To restore an AWS EC2 Windows instance from an AMI backup while ensuring it retains the same IP address (198.162.0.1) and other settings, follow these detailed steps. This guide covers:

1. Creating and managing AMI backups
2. Restoring an instance from an AMI
3. Ensuring the restored instance retains the same IP address and application environment

### Scenario:
- **Current instance**: A Windows EC2 instance running in a private subnet with IP `198.162.0.1`.
- **Issue**: After manually applying Windows patches, the application stopped working, or the server went down and isn't coming back up.
- **Solution**: Restore the instance using a previously created AMI, while maintaining the original configuration, including the private IP.

### Pre-requisite Steps: AMI Backup (for future reference)
If you haven't already created an AMI backup, make sure to do so periodically, especially before performing any patching or maintenance. Here's how you can create an AMI of your instance:

1. **Create AMI Backup**:
   - Go to the **AWS Management Console** > **EC2 Dashboard** > **Instances**.
   - Select the instance you want to back up.
   - Click **Actions** > **Image and templates** > **Create Image**.
   - Name the AMI appropriately (e.g., `pre-patch-backup`).
   - Leave other settings as default (e.g., No Reboot unless you need specific configurations).
   - Click **Create Image**.

Once created, this AMI will contain the complete snapshot of your EC2 instance, including the operating system, installed software, and data.

---

### Restoring the EC2 Instance from AMI

If your Windows EC2 instance is no longer working after patching, follow these steps to restore the instance to its pre-patched state using the AMI backup:

#### Step 1: Stop or Terminate the Current Instance
To ensure there's no conflict with IP assignment, first stop or terminate the current faulty instance.

1. Go to the **AWS EC2 Console**.
2. Locate the faulty EC2 instance.
3. **Stop** or **Terminate** the instance:
   - Stopping will preserve the data but will stop the service.
   - Termination will remove the instance, but your EBS volumes will remain intact.

#### Step 2: Launch a New Instance from AMI

1. In the **EC2 Console**, go to **AMIs** under **Images** in the sidebar.
2. Select the AMI you created before applying the patch.
3. Click **Launch Instance from AMI**.
4. **Instance Configuration**:
   - Select the same **Instance Type** that the previous instance was running on.
   - Choose the **same VPC** and **private subnet** where the original instance was running.
   
#### Step 3: Assign the Same Private IP Address
By default, AWS assigns a new private IP to any newly launched instance. Since your application needs the same private IP (`198.162.0.1`), you need to configure it manually.

1. In the **Network Settings** section, click on **Edit**.
2. Under the **Network interfaces** section, choose the private subnet in which the original instance was deployed.
3. **Manually assign the private IP address**:
   - In the **Primary IP** field, type `198.162.0.1`.
   - Ensure no other instance in the subnet is using this IP address.

   > **Note**: You can only assign the private IP manually when launching the new instance. If this field is left blank, AWS will assign a random IP automatically.

4. Configure the **same security groups** and **key pair** that were used by the original instance.

5. **Storage configuration**:
   - Ensure the EBS volumes attached to the new instance match those from the original instance. You can configure this by copying volume settings from the AMI or attaching previous EBS volumes manually if needed.

6. Review all settings and **Launch** the instance.

#### Step 4: Check the Application and Environment

Once the instance is launched, connect to it using RDP to verify that everything is working as expected:

1. **Connect via RDP**:
   - Use the same private IP (`198.162.0.1`) to access the instance using an RDP client.
   - Ensure you are using the same key pair used during the instance launch.

2. **Verify Application**:
   - Check the application or services that were previously running on the instance.
   - Ensure that the application is functioning as expected without the issues introduced by the Windows patching.

#### Step 5: Restore or Re-Attach Additional Resources (Optional)

If the previous instance had additional resources (like EBS volumes, elastic IPs, etc.), ensure those are restored or attached to the new instance.

1. **Re-attach EBS volumes**:
   - If you want to use specific volumes from the terminated instance, go to **Volumes** in the EC2 Console and attach them to the new instance.
   - Ensure you mount the volumes to the correct paths within the instance.

2. **Elastic IP**:
   - If the previous instance had an Elastic IP (EIP), associate the same EIP to the new instance.

3. **Elastic Load Balancer**:
   - If your application is behind an ELB, ensure the new instance is registered to the ELB.

---

### Verifying the Restoration

Once the instance has been restored and configured with the same IP address and environment, it's important to:

- Test the functionality of the application.
- Ensure all necessary network configurations, like security groups and VPC routing, are properly configured.
- Confirm that the IP address (`198.162.0.1`) has been successfully reassigned and that no conflicts are present in the private subnet.

---

### Preventing Future Issues
- **Regular AMI Backups**: Before any future patching or maintenance, create an AMI backup.
- **Automated Patch Management**: Consider setting up **AWS Systems Manager** for automated patch management and rollback options.
- **Monitoring and Alerts**: Use AWS CloudWatch to set up monitoring for instance health and create alarms that notify you of service failures in real-time.

This process ensures you can safely restore your Windows EC2 instance to its original state, including its IP address and application environment, using an AMI backup.
