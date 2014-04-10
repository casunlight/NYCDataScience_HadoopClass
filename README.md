# Hadoop Workshop I and II Tutorial:
## RSVP for the class from [NYC Data Science Academy](http://nycdatascience.com/course/hadoop-data-analytic-platform/)
   * Take it online (17 weeks long, 2 hours/week)
   * Take it in classroom on workday(5 weeks, 10 of Wed and Fri, 3.5 hours/night)
   * Take it in classroom on weekend(5 weeks, 5 full Sundays, 7 hours/day)

## Goal: 

	* configure a Hadoop Cluster on Amazon EC2

	* run two sample map reduce jobs on the cluster

## I will go over the details:

	+ Preparation

	+ Server configuration

	+ Hadoop installation and configuration

### Preparation:

1. apply for a AWS acct 
    - go to http://aws.amazon.com/account/
	- put name, email, click on "sign in";
	- put full name, email, billing info, credit bard;
	- pass the phone number authorization;
	- select AWS support plan, use Basic (Free);
	- see message "Thank you for updating your Amazon Web Services Account!”.

2. log in your acct
    - sign in using the acct/pass you just made.
	- choose "Launch the AWS Management Console";
	- click on button "sign in to the AWS console";
	- choose "I am a returning user and my password is";
	- click "sign in using our secure server";
	- see "EC2 Dashboard" on the top left corner.

3. create your key pair
   - you authorize yourself and communicate with Amazon cloud using public and private key pair.
   - on your left panel of EC2 dashboard, click on 
	"network & security" category's "Key Pairs";
   - click on button "create key pair";
   - type your key pair name, for me, I put "EC2UbuntuLTSThreeT1Micro";
   - see a download file window, choose a secure folder to save your .pem file which will be your private key. For me, I get the file called "EC2UbuntuLTSThreeT1Micro.pem";

4. create EC2 instances
   - How to understand instance? each instance is like a seperate machine(or pc or laptop).
   - on your left panel of EC2 dashboard,, click on 
	"instances" category's "instances";
   - click on button "launch instance";
	choose the system, find the 9th options--"Ubuntu Server 12.04 LTS (HVM) - ami-5fa4a236  64 bit";
   - click on "select";
   - keep the default setting, choose "next:configure instance details";
   - put "3" into "Number of instances";
   - click on "review and launch";
   - click on "launch";
   - choose the key pair "EC2UbuntuLTSThreeT1Micro";
   - check the acknowleage box;
   - click on "launch instance",you should see "Your instances are now launching";
    scroll down to the bottom, click on "view instances",click one each instance and rename them, such as "meetup1","meetup2","meetup3",wait till you see the status checks is changed from "initializing" to "2/2 checks passed".

5. configure security group
	- each group is like a firewall. The nodes of the same cluster need to be in the same security group.
	- click on "create security groups",put name such as "launch-wizard-1" or change to your customized name;
	- under "inbound" tab, cick on add rules
      - add five rules:
        - choose type="SSH",  "save";
		- choose type="ALL ICM", Source=Anywhere, "save";
		- choose type="Custom TCP", port range="9000", choose "anywhere",save;
		- choose type="Custom TCP", port range="9001", choose "anywhere",save;
		- choose type="Custom TCP", port range="50000 - 50100", choose "anywhere", save;

		```no-highlight
		double check all your inbound setting, they should look like:

		| Type            | Protocol  | Port Range      | Source            |
		| ----------------|:---------:| ---------------:|------------------:|
		| SSH             | TCP       | 22              |Anywhere 0.0.0.0/0 |
		| Custom TCP Rule | TCP       | 9001            |Anywhere 0.0.0.0/0 |
		| Custom TCP Rule | TCP       | 9000            |Anywhere 0.0.0.0/0 |
		| Custom TCP Rule | TCP       | 50000 - 50100   |Anywhere 0.0.0.0/0 |
		| All ICMP        | ICMP      | 0 - 65535(or na)|Anywhere 0.0.0.0/0 |

6. manage your instance
   - know how to turn on and off the instances
   - select the instance, right click to choose "stop", you won't be able to use this instance and won't be charged;
   - select the instance, right click to choose "start", you can use the instance and will be charged.
   --next time, if you restart the instance, your "public DNS" will be different, but your "private DNS" will not changed.  If you reboot the instance, your "public DNS" will be the same as before rebooting.

### Server configuration

1. You are required to use vi editor. The basic operations are:   

	```no-highlight
	 Cheet sheet for vi

    | what you want to do  | Key stokes       | 
    | ---------------------|:----------------:| 
    | insert/edit model    | i                | 
    | finish and save      | ESC->:->wq->Enter|
    | finish and not save  | ESC->q!->Enter   |
    | go to the end of line| ESC->o           |

2. generate your server rsa key for three instances
 - find your meetup1's public DNS and get its RSA public key
    - open terminal 1, ssh to your server
      - before you run ssh, make sure you are at the location you can access your .pem file. I save .pem in my ".ssh" folder, so I do "cd .ssh" first.
   	  - remote access to your instance by "ssh  -i EC2UbuntuLTSThreeT1Micro.pem  ubuntu@eyour_public_dns". 

	  ```no-highlight
       make your own reference table 

  	 | machine name| public DNS                             | 
  	 | ------------|:--------------------------------------:|  	  
  	 | meetup1     |ec2-54-86-2-169.compute-1.amazonaws.com |
  	 | meetup2     |ec2-54-86-4-68.compute-1.amazonaws.com  |
  	 | meetup3     |ec2-54-86-10-200.compute-1.amazonaws.com|

  	- you should be able to make your assemblied commands:
      - ssh  -i EC2UbuntuLTSThreeT1Micro.pem  ubuntu@ec2-54-86-2-169.compute-1.amazonaws.com
      - ssh  -i EC2UbuntuLTSThreeT1Micro.pem  ubuntu@ec2-54-86-4-68.compute-1.amazonaws.com
      - ssh  -i EC2UbuntuLTSThreeT1Micro.pem  ubuntu@ec2-54-86-10-200.compute-1.amazonaws.com

   	- Are you sure you want to continue connecting (yes/no)? yes
    - generate your server key
    - ssh-keygen -t rsa 
    - do "Enter" three times
    - cd .ssh
    - vi id_rsa.pub
    - copy and paste it into a text file for future

 - find your meetup2's public DNS and generate its RSA public key
    - open terminal 2, ssh to your server
    - two parts are different from meetup1
      - meetup2 has different public DNS address
      - copy and paste meetup2's id_ras.pub into the same file as second line
 - find your meetup3's public DNS and generate its RSA public key
     - open terminal 3, ssh to your server
     - two parts are different  from meetup1
      - meetup3 has different public DNS address
      - copy and paste meetup3's id_ras.pub into the same file as third line
 - in the end, you should have a file which contains three rsa keys

3. configure "authorized_keys" file for three instances
 - on meetup1 instance
  - cd .ssh
  - vi authorized_keys
  - go to the end of file by "ESC->o"
  - copy/paste three rsa keys to the end of file
    you should have one original keys and three new keys in the end
  - save and exit file by "ESC->wq!"
 - on meetup2 instance, do the same
 - on meetup3 instance, do the same

4. configure "hosts" file for three instances
 - on meetup 1 instance
 	- sudo vi /etc/hosts
 	- find your private DNS from EC2 console
 	-- add three lines to the end of files, such as
 	172.31.43.177 meetup1
 	172.31.43.179 meetup2
 	172.31.43.178 meetup3
 - on meetup2 instance, do the same
 - on meetup3 instance, do the same

5. test connections among cluster
 - from meetup1
  	- ping meetup2
  	- ping meetup3
 - from meetup2
  	- ping meetup1
  	- ping meetup3
 - from meetup3
  	- ping meetup1
  	- ping meetup2
 - ctrl +c to stop pinging

6. install Java
 - run the comands for each instance
	- sudo add-apt-repository ppa:webupd8team/java
	- sudo apt-get update
	- sudo apt-get install oracle-java7-installer
	(do "Enter" to select "ok", user cursor to select second "ok" and enter again)
	- sudo apt-get install oracle-java7-set-default
	- exit
	- ssh  -i EC2UbuntuLTSThreeT1Micro.pem  ubuntu@ec2-54-209-171-193.compute-1.amazonaws.com
	(logout and login again to validate your new configuration)
	- echo $JAVA_HOME
	(you should see "/usr/lib/jvm/java-7-oracle")

### Hadoop installation and configuration
 - We are using stable Hadoop version 1.2.1 (2014-04-04)
 - the mirror is from [Columbia Univ](http://mirror.cc.columbia.edu/pub/software/apache/hadoop/common/hadoop-1.2.1/)
 - all the operations in the below will be run on meetup1(your master node)

 1. download haoop source codes
	- wget http://mirror.cc.columbia.edu/pub/software/apache/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz
	- tar -xzvf hadoop-1.2.1.tar.gz 
	- cd hadoop-1.2.1/conf 
 2. configure the environment
	- configure Java Path
	  - vi hadoop-env.sh
	  - search for the line "# The java implementation to use.  Required." 
	  - delete the "#" 
	  - set "export JAVA_HOME=/usr/lib/jvm/java-7-oracle"
	  - save the file
	- configure core-site file
	  -vi core-site.xml 
	  - between <configuration> and </configuration>, put
		  <property>
			<name>fs.default.name</name>
				<value>hdfs://meetup1:9000</value>
		  </property>

		  <property>
			<name>hadoop.tmp.dir</name>
			<value>/home/ubuntu/hadoop/tmp</value>
		  </property>
	- make new folder "~/hadoop/tmp"
	  - cd ~
	  - mkdir hadoop
	  - cd hadoop
	  - mkfir tmp
    - configure redundance, the value is usually set as 1 or 2 and less than total number of slave nodes.
      - cd ~
      - cd hadoop-1.2.1/conf
      - vi hdfs-site.xml
      - between <configuration> and </configuration>, put
      	<property>
			<name>dfs.replication</name>
			<value>2</value>
		</property>
	- configure master file
	  - delect localhost
	  - put meetup1
	- configre slave file
	  - delect localhost
	  - put
	      meetup2
	      meetup3
	- copy all the configuration from master node to two slave nodes
	  - scp -r ~/hadoop-1.2.1 meetup2:/home/ubuntu
      - scp -r ~/hadoop-1.2.1 meetup3:/home/ubuntu

 3. format
	- ~/hadoop-1.2.1/bin/hadoop namenode -format 
	- you should get "Storage directory /home/ubuntu/hadoop/tmp/dfs/name has been successfully formatted."
	- cd ~/hadoop/tmp/
	- you should find the two folders 
	  - dfs->name->current
	  - dfs->name->image

 4. start your hadoop
  - ~/hadoop-1.2.1/bin/start-all.sh
  - test whether the hadoop is running
  - run "jps" on three instances
  - on your master node, you should see (the numbers will vary)
  	6919 NameNode
	7237 JobTracker
	7155 SecondaryNameNode
	7445 Jps
  - on two slave nodes, you should see (the numbers will vary)
    7653 TaskTracker
	7490 DataNode
	7713 Jps

 5. stop your hadoop
  - ~/hadoop-1.2.1/bin/stop-all.sh 

## Congratulation! You have your first hadoop cluster!

 ### Extra note on your hadoop log
  - you can find log files under "cd ~/hadoop/tmp/mapred/local/userlogs/"
  - pick one job folder, such as "job_201404071413_0002"
  - pick one log file, such as "vi attempt_201404071413_0002_m_000001_3"

