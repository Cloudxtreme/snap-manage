**Purpose of the scripts**

The scripts are designed to automate the daily and monthly creation of snapshots as well as clean up old snapshots to avoid any manual management of large numbers of snapshots.

**Requirements**

	Linux EC2 instance (recommended t2.micro amazon linux)
	Latest lib boto
	EC2 role for granting snapshot management access

**Setup**

**EC2 Role**

The purpose of this role is to allow the EC2 server associated with it access to create and delete snapshots without having keys generated and stored in the scripts. The EC2 Role must be associated at instance creation time otherwise the instance will not have access to the resources required. Create a role with the following custom IAM Role and policy.

	{
	  "Statement": [
	    {
	      "Sid": "Stmt1407157182165",
	      "Action": [
	        "ec2:CopySnapshot",
	        "ec2:CreateSnapshot",
	        "ec2:DeleteSnapshot",
	        "ec2:DescribeSnapshotAttribute",
	        "ec2:DescribeSnapshots",
	        "ec2:ModifySnapshotAttribute",
	        "ec2:ResetSnapshotAttribute"
	      ],
	      "Effect": "Allow",
	      "Resource": "*"
	    },
	    {
	      "Sid": "Stmt1407159908122",
	      "Action": [
	        "ec2:CreateTags",
	        "ec2:DeleteTags",
	        "ec2:DescribeTags"
	      ],
	      "Effect": "Allow",
	      "Resource": "*"
	    },
	    {
	      "Sid": "Stmt1407159989326",
	      "Action": [
	        "ec2:DescribeVolumeAttribute",
	        "ec2:DescribeVolumeStatus",
	        "ec2:DescribeVolumes",
	        "ec2:DescribeVolumeStatus"
	      ],
	       "Effect": "Allow",
	        "Resource": "*"
	    }
	  ]
	}

**EC2 Instance**

**Instance Type**

The code has been tested on Amazon Linux

	Recommended: t2.micro
	AMI ID: amzn-ami-hvm-2014.03.2.x86_64-ebs (ami-892fe1fe)
	Launch the instance with the ec2-role defined above

**Upgrading boto**

On Amazon Linux you need to install pip to perform the upgrade:

	sudo yum install python-pip
	sudo pip install boto --upgrade

This will mean you are running the latest version of boto which should ensure compatibility with the scripts. 

**Scripts**

All scripts can be found in this github repository:

	https://github.com/ngineered/snap-manage

The snap-manage script should be placed in:

	/usr/sbin/

and make sure its executable:

	chmod 755 /usr/sbin/snap-manage

**Example Usage**

To run the scripts from the command line make sure you have /usr/sbin in your $PATH

*For help*

	snap-manage -h

*Example of a daily backup that clears after 14 days*

	snap-manage -a 624900703163 -s Daily -v 14

*Same example but in a different region to eu-west-1*

	snap-manage -a 624900703163 -r us-west-1 -s Daily -v 14

*Monthly snapshot example*

	snap-manage -a 624900703163 -s Monthly -v 90

**Cron**

Example cron to run monthly and daily backups from /etc/cron.d/snapshots

	00 01 * * * ec2-user /usr/sbin/snap-manage -a 624900703163 -s Daily -v 14
	00 03 30 * * ec2-user /usr/sbin/snap-manage -a 624900703163 -s Monthly -v 90
