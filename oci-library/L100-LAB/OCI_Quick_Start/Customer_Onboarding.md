# Customer on boarding lab - VCN, Compute and Block Volume
  
## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Recommended Learning Assets](#recommended-learning-assets)

[Practice 1: Sign in to OCI Console and create VCN](#practice-1-sign-in-to-oci-console-and-create-vcn)

[Practice 2: Creat ssh keys, compute instance and Block Volume](#practice-2-creat-ssh-keys,-compute-instance-and-block-volume)

[Practice 3: Install httpd on compute instance and install an app on Block Volume](#practice-3-install-httpd-on-compute-instance-and-install-an-app-on-block-volume)

[Practice 4: Detach the block volume and launch second compute instance using boot volume and attach Block Volume](#practice-4-detach-the-block-volume-and-launch-second-compute-instance-using-boot-volume-and-attach-block-volume)

[Practice 5: Delete the resources](#practice-5-delete-the-resources)

[Appendix A: ISCSI Method to Attach Block Volume to Compute Instance](#appendix-a-iscsi-method-to-attach-block-volume-to-compute-instance)

[Appendix B: ISCSI Method to Detach block volume](#appendix-b-iscsi-method-to-detach-block-volume)



## Overview

In this lab we will create a compute instance, install httpd server on it, attach block volume to it and install an app on the Block Volume
We will then create a second compute instance using the boot volume of the first compute instance and move the Block Volume to 
the second compute instance (with the app installed).
The goal of the lab is to demonstrate launching VCN, Compute Instance, using Boot volume of compute instance and 
how easy it is to move additional storage with applicatons/tools installed between compute instances

## Pre-Requisites

- Oracle Cloud Infrastructure account credentials (User, Password, Tenant, and Compartment)  

## Recommended Learning Assets
1. OCI Training : https://cloud.oracle.com/en_US/iaas/training

2. Familiarity with OCI console: https://docs.us-phoenix-1.oraclecloud.com/Content/GSG/Concepts/console.htm

3. Overview of Networking: https://docs.us-phoenix-1.oraclecloud.com/Content/Network/Concepts/overview.htm

4. Familiarity with Compartment: https://docs.us-phoenix-1.oraclecloud.com/Content/GSG/Concepts/concepts.htm

5. Connecting to a compute instance: https://docs.us-phoenix-1.oraclecloud.com/Content/Compute/Tasks/accessinginstance.htm


## Practice-1: Sign in to OCI Console and create VCN


**Note:** OCI UI is being updated thus some screenshots in the instructions might be different than actual UI

**Before You Begin**

- We recommend using Chrome or Edge as the broswer. Also set your browser zoom to 80%

1. Sign in using your tenant name, user name and password.

2. Once signed in select the compartment assigned to you from drop down menu on left part of the screen

3. From the OCI Services menu,click **Virtual Cloud Network** under Networking and click **Create Virtual Cloud Network**

**NOTE:** Ensure the correct Compartment is selected under COMPARTMENT list

![]( img/RESERVEDIP_HOL001.PNG)
![]( img/RESERVEDIP_HOL002.PNG)

4. Fill out the dialog box:

- **Create in Compartment:** Has the correct compartment

- **Name:** Enter easy to re¬member name

- **Create Virtual Cloud Network Plus Related Resources:** Select this option.

- Click **Create Virtual Cloud Network**

- Click **Close**

![]( img/RESERVEDIP_HOL003.PNG)
![]( img/RESERVEDIP_HOL004.PNG)

5. We will open port 80 on this VCN to provide http access to app on the compute instance(to be installed later on. Click **Security List** and then **Default Security list for<YOUR_VCN_NAME>**

![]( img/Customer_Lab_001.PNG)

6. In Security list details page, Click **Edit All Rules** and Click **+Another Ingress Rule** under 
Allow Rules for Ingress and add below rule:
- Make sure STATELESS Flag in un-checked
- SOURCE TYPE: CIDR
- SOURCE CIDR: 0.0.0.0/0
- IP PROTOCOL: TCP
- SOURCE PORT RANGE: ALL
- DESTINATION PORT RANGE: 80

![]( img/Customer_Lab_002.PNG)

7. Click **Save Security List Rules** at the bottom
              
## Practice 2: Creat ssh keys, compute instance and Block Volume

1. Click the Apps icon in the toolbar and select  Git-Bash to open a terminal window.

![]( img/RESERVEDIP_HOL006.PNG)

2. Enter command 
```
ssh-keygen
```
**HINT:** You can swap between OCI window, 
git-bash sessions and any other application (Notepad, etc.) by clicking the Switch Window icon 

![]( img/RESERVEDIP_HOL007.PNG)

3. Press Enter When asked for ‘Enter File in which to save the key’, ‘Created Directory, ‘Enter passphrase’, and ‘Enter Passphrase again.

![]( img/RESERVEDIP_HOL008.PNG)

4. You should now have the Public and Private keys:

/C/Users/ PhotonUser/.ssh/id_rsa (Private Key)

/C/Users/PhotonUser/.ssh/id_rsa.pub (Public Key)

**NOTE:** id_rsa.pub will be used to create 
Compute instance and id_rsa to connect via SSH into compute instance.

**HINT:** Enter command 
```
cd /C/Users/PhotonUser/.ssh (No Spaces) 
```
and then 
```
ls 
```
to verify the two files exist. 

5. In git-bash Enter command  
```
cat /C/Users/PhotonUser/.ssh/id_rsa.pub
```
 , highlight the key and copy 

![]( img/RESERVEDIP_HOL009.PNG)

6. Click the apps icon, launch notepad and paste the key in Notepad (as backup)

![]( img/RESERVEDIP_HOL0010.PNG)

7. Switch to the OCI console. From OCI servies menu, Click **Instances** under **Compute** 

8. Click Create Instance. Fill out the dialog box:

- **Name:** Enter a name 

- **Availability Domain:** Select availability domain

- **Image Operating System:** For the image, we recommend using the Latest Oracle Linux available.

- **Choose Instance Type:** Select Virtual Machine

- **Choose Instance Shape:** Select VM shape

- **Configure Boot Volume:** Leave the default

- **Add SSH Keys:** Choose ‘Paste SSH Keys’ and paste the Public Key saved earlier.

- **Virtual Cloud Network Compartment:** Choose your compartment

- **Virtual Cloud Network:** Select the VCN you created in the previous section. 

- **Subnet Compartment:** Choose your compartment. 

- **Subnet:** Choose the first Subnet

9. Click **Create**

**NOTE:** If 'Service limit' error is displayed choose a different shape such as VM.Standard.E2.2 OR VM.Standard2.2 OR choose a different AD
![]( img/RESERVEDIP_HOL0011.PNG)

10. Wait for Instance to be in **Running** state. In git-bash Enter Command:
```
 cd /C/Users/PhotonUser/.ssh
```
11. Enter **ls** and verify id_rsa file exists

12. Enter command 
```
ssh –i id_rsa opc@<PUBLIC_IP_OF_COMPUTE_INSTANCE>
```
**NOTE:** User name is opc

**HINT:** If ‘Permission denied error’ is seen, ensure you are using ‘-i’ in the ssh command

13. Enter ‘Yes’ when prompted for security message

![]( img/RESERVEDIP_HOL0014.PNG)
 
14. Verify opc@<COMPUTE_INSTANCE_NAME> appears on the prompt

15. From OCI services menu Click **Block Volumes** under Block Storage, then Click **Create Block Volume**.

16. Fill out the dialog box: 
- **Create in Compartment:** Has the correct compartment selected.
- **Name:** Enter a name for the block volume (e.g. "block_vm).
- **Availability Domain:** Select the first available domain (must be same as 
Compute).
- **SIZE:** Set to 50
- **BACKUP POLICY:** Set to None (If this field shows ‘Error Retrieving Value’ then leave it as is)
17. Click **Create Block Volume**. Wait for volume to become available. Wait for Block Volume state to change from 'Provisioning' to 'Available'

![]( img/Customer_Lab_003.PNG)

18. Attach Block volume to your compute instance. From OCI services menu Click **Instance** under Compute 

19. For the compute instance created earlier click  Action item. Click **Attach Block Volume**.

![]( img/Customer_Lab_004.PNG)

20. Fill out the dialog box:
- Choose how you want to attach your block volume:Check Paravirtualized

**NOTE:** We can also use ISCSI mode, in which case we will not have to use ISCSI commands as detailed later on. For more information please refer to ;
**https://docs.cloud.oracle.com/iaas/Content/Block/Concepts/overview.htm#attachtype** OR refer Appendix section at the end of the lab.
- BLOCK VOLUME COMPARTMENT: Ensure correct compartment is selected
- Block Volume: Choose the volume created earlire
- Access: Choose READ/WRITE

21. Click **Attach**.

22. Click **Close**.

**We now have a block volume created and attached to the Compute instance.**

23. Verify the Block Volume is attached by clicking the Compute Instance name.

![]( img/Customer_Lab_005.PNG)


## Practice 3: Install httpd on compute instance and install an app on Block Volume

1. Switch to ssh session to compute install. Install httpd server, Enter Command:
```
sudo yum -y install httpd 
```
2. Configure Compute instance firewall, Enter commands:
```
sudo firewall-cmd --permanent --add-port=80/tcp `
```
(Open port 80 on the firewall to allow http and https traffic).

**NOTE:** Despite the line wrapping, the --add-port flag has no spaces.
```
sudo firewall-cmd --reload 
```
(Reload the firewall to activate the rules).

3. Start httpd, Enter command:
```
sudo systemctl start httpd 
```
4. To format the block volume, Enter Command: 
```            
sudo fdisk /dev/<VOLUME_NAME> -l 
```
**For example sudo fdisk /dev/sdb -l**   
Wait for formatting to complete

5. Create a file system on the block volume, Enter Command: 
```
sudo mkfs.ext4 -L datapartition /dev/<VOLUME_NAME>   
```         
This will create the file system on the entire disk. Enter y when prompted with 'Proceed anyway'

**NOTE:** For this lab we will not be creating additional partitions

6. Create a directory where the disk will be mounted,Enter commands:
```
sudo mkdir -p /mnt/www/html            
```

7. Mount the disk to the newly created directory,
Enter command:
```
sudo mount  /dev/<VOLUME_NAME>  /mnt/www/html
```

8. Verify /dev/<VOLUME_NAME> volume is now mounted to /mnt/www/html directory, Enter command 
```
lsblk 
```
(in this case the volume is called sdc)
![]( img/Customer_Lab_006.PNG)

9. Next we will download an app and install it. Enter command:
```
cd /home/opc
```

10. Enter Command:
```
wget https://github.com/snafuz/oci-quickstart-lab/archive/master.zip
```

11. Enter Command: 
```
unzip master.zip
```

12. Enter Command:
```
sudo cp -R oci-quickstart-lab-master/static/* /mnt/www/html/
```

13. Modify httpd.conf file, Enter Command:
```
sudo vi /etc/httpd/conf/httpd.conf (for vi) 
```
OR 
```
sudo nano  /etc/httpd/conf/httpd.conf  (for nano)
```

**NOTE:** in vi you can save the file after modification by using Esc :wq 

14. Search for string /var/www and replace it with /mnt/www/html . This will be done in 3 locations

![]( img/Customer_Lab_007.PNG)

15. Save and Exit (in vi user Esc :wq)

16. Enter command:
```
sudo chcon -R --type=httpd_sys_rw_content_t /mnt
```

17. Restart httpd server, Enter command:
```
sudo systemctl restart httpd 
```

18. Launch a web browser and Enter compute Instance's public IP,
```
http://<COMPUTE_INSTANCE_PUBLIC_IP>
```

19. Screen like below should appear.

![]( img/Customer_Lab_008.PNG)

**We have initialized httpd.conf file. Next we will create a second compute instance using the boot volume of the first compute instance and attach Block Volume to it**

## Practice 4: Detach the block volume and launch second compute instance using boot volume and attach Block Volume

**In this section we will detach the block volume, Stop the compute instance, use its boot volume to launch a second compute instnace and delete the second compute instance**

1. In ssh session to the compute instance unmount the directory from block volume, Enter command:
```
sudo umount /dev/<VOLUME_NAME> 
```
2. In OCI console window, click your compute instance name and in **Attached Block Volume** section  click the action icon and **click Detach**

![]( img/Customer_Lab_009.PNG)

3. Stop your compute instance by Clicking **Stop** in compute instance details page and then **OK** in Confirm window

![]( img/Customer_Lab_010.PNG)

4. Once the instance is in Stopped state, click **Boot Volume**, Click action icon and click **Detach**. Click **OK** in Confirm window.

![]( img/Customer_Lab_011.PNG)

5. Once the Boot volume is detached, Click **Terminate** to Terminate the instance.

6. Click the action icon, click **View Boot Volume Details**.

![]( img/Customer_Lab_012.PNG)

7. In the Boot Volume Details window click **Create Instance:*
- Name: Enter a name (e.g. "boot volume instance").
- Availability Domain: Make sure its same AD as where the block volume was created

**NOTE :** Boot Volume field is set to BOOT VOLUME and to the boot volume you detached from the original Instance.

- Image Operating System: Use the Latest Oracle Linux available.
- 7.4 Intance Type: Select Virtual Machine
- Instance Shape: Choose the shape with least OCPU

**NOTE:** Here you can choose a larger/smaller shape which can be used to Scale a compute instance

- SSH Keys: Choose ‘Paste SSH Keys’ and paste the Public Key saved earlier.
- Virtual Cloud Network Compartment: Select yor compartment 
- Virtual Cloud Network: Select the VCN created earlier
- Subnet compartment: Select yor compartment 
- Subnet: Select the Public subnet.

8. Click Create Instance.
**NOTE:** If 'Service limit' error is displayed choose a different shape such as VM.Standard.E2.2 OR VM.Standard2.2 OR Choose a different AD
![]( img/Customer_Lab_013.PNG)
![]( img/Customer_Lab_014.PNG)

9. Once the instance is in Running state, attach the block volume to this new instance using OCI Console.**Ensure to use Paravirtualized mode.**

10. Mount the block volume as before, Enter Command:
```
sudo mount  /dev/<VOLUME_NAME>  /mnt/www/html
```

11. Restart httpd, Enter command
```
sudo systemctl restart httpd
```

12. Launch a web browser and Enter compute Instance's public IP,
**http://<COMPUTE_INSTANCE_PUBLIC_IP>**

13. Screen like below should appear, Enter the required information
![]( img/Customer_Lab_015.PNG)

**We have now successfully launched a compute instance using another instance's boot volume and re-attached a block volume that was attached to another instance. The block volume preserved all the data on it during this process. Moreover the compute instance launched using the boot volume had httpd server and fire wall configuration already present**

## Practice 5: Delete the resources

1. Switch to  OCI console window

2. If your Compute instance is not displayed, From OCI services menu Click Instances under Compute

3. Locate first compute instance, Click Action icon and then **Terminat** 

![]( img/RESERVEDIP_HOL0016.PNG)

4. Make sure Permanently delete the attached Boot Volume is checked, Click Terminate Instance. Wait for instance to fully Terminate

![]( img/RESERVEDIP_HOL0017.PNG)

5. From OCI services menu Click **Block Volumes** under Block Storage

6. Find the storage block volume you created.

**HINT:** If multiple storage block volumes are listed, scroll down to find the one you created.   

7. Click the Action icon and select **Terminate**

8. Click OK in the confirmation window.

![]( img/Customer_Lab_016.PNG)

9. From OCI services menu Click **Virtual Cloud Networks** under Networking, list of all VCNs will 
appear.

10. Locate your VCN , Click Action icon and then **Terminate**. Click **Delete All** in the Confirmation window. Click **Close** once VCN is deleted

![]( img/RESERVEDIP_HOL0018.PNG)

11. From OCI services menu Click **Networking**, then **Public IPs**,locate the Reserved Public IP you created. Click Action icon and then **Terminate**

![]( img/RESERVEDIP_HOL0019.PNG)

***Congratulations! You have successfully completed the lab. ***

## Appendix A: ISCSI Method to Attach Block Volume to Compute Instance

1. Click Action icon for the Block Volume, then **ISCSI Commands & Information**

**NOTE:** The iSCSI Commands and Information dialog box displays specific identifying information about your volume and the iSCSI commands you'll need. The commands are ready to use with the appropriate information included. You can copy and paste the commands into your instance once you login.

![]( img/Customer_Lab_017.PNG)

2. Click **Copy** in **ATTACH COMMANDS** section. 

![]( img/Customer_Lab_018.PNG)

3. Click **Close** to close the window 

4. In OCI Console Window, click the Apps icon and click Notepad. 

**HINT:** You can swap between the OCI window and any other application (Notepad etc.) by clicking the Switch Window icon

5. Paste the ISCSI commands in Notepad using your mouse/touch pad or Ctrl v. 

**To attach ssh to the compute instance and paste the ISCSI commands copied earlier**

## Appendix B: ISCSI Method to Detach block volume

1. Click Action icon for the Block Volume, then **ISCSI Commands & Information**

2. Click **Copy** in **DETACH COMMANDS** section. 

![]( img/Customer_Lab_019.PNG)

3. Paste the detach command in the ssh session to the compute instance

4. In OCI console window, click your compute instance name and in **Attached Block Volumes** section click the action icon and click **Detach**

![]( img/Customer_Lab_020.PNG)