## How to create an EC2 instance
use 'Deep Learning AMI (Amazon Linux) Version 8.0' as an example

1. Go to AWS EC2 instance homepage

https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Home:


choose the area (eg, us-east-2a) you want to launch the instance, then click launch and select an instance on the webpage below. I chose 'Deep Learning AMI (Amazon Linux) Version 8.0' for this time.
 
https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#LaunchInstanceWizard:
 

2. Choose an instance type. Based on your need you can choose CPU or GPU based instance. eg, p2x.large

![alt-text-1](ec2img/step2.jpg)


3. Confiuge instance details
   
   set up vpc as ```vpc-870644ee```
   
   set up subnet as ```subnet-18c57cb71``` for us-east-2a

4. Add storage or attach volume later 

5. Add tags. Remember to give a name to your instance.

6. Configue security group. Choose the ```Research Development``` group.

7. Review and launch the instance.



## Attach and mount an EBS volume to EC2 linux instance
   
   Create EBS volume, follow the instuction below
 
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html
 
1. Head over to EC2 –> Volumes and create a new volume of your preferred size and type.

2. Select the created volume, right click and select the “attach volume” option.

3. Select the instance from the instance text box as shown below.

   Attach EBS volume

4. Now, login to your ec2 instance and list the available disks.
```
east2bastion
ssh ec2-user@xxxxxxx
lsblk
```
The above command will list the disk you attached to your instance.

5. Check if the volume has any data using the following command.
```
sudo file -s /dev/xvdf
```
If the above command output shows ```/dev/xvdf: data```, it means your volume is empty.

6. Format the volume to ext4 filesystem  using the following command.
```
sudo mkfs -t ext4 /dev/xvdf
```

7. Create a directory of your choice to mount our new ext4 volume. 
```
sudo mkdir /mnt/data/
```

8. Mount the volume to ```/mnt/data``` directory using the following command.
```
sudo mount /dev/xvdf /mnt/data/
```

9. cd into ```/mnt/data``` directory and check the disk space for confirming the volume mount.
```
cd /mnt/data
df -h .
```

10. Permanently mount a volume
```
sudo cp /etc/fstab /etc/fstab.orig
```
and add following info to ```/etc/fstab``` by vim.
 ```
 /dev/xvdf  /mnt/data   ext4    defaults,nofail
 ```
 
11. Remove sudo
```
sudo chown ec2-user /mnt/data
```
