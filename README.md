Hadoop Workshop I and II Tutorial:

Goal: 

— configure a Hadoop Cluster on Amazon EC2

— run two sample map reduce jobs on the cluster

I will go over the details:

- Preparation

- Server configuration

- Hadoop installation and configuration

Preparation:

1. apply for a AWS acct 
   * go to http://aws.amazon.com/account/
	- put name, email, click on "sign in";
	- put full name, email, billing info, credit bard;
	- pass the phone number authorization;
	- select AWS support plan, use Basic (Free);
	- see message "Thank you for updating your Amazon Web Services Account!”.

2. log in your acct
   * sign in using the acct/pass you just made.
	- choose "Launch the AWS Management Console";
	- click on button "sign in to the AWS console";
	- choose "I am a returning user and my password is";
	- click "sign in using our secure server";
	- see "EC2 Dashboard" on the top left corner.

3. create your key pair
   * you are authorizate yourself and communicate with Amazon cloud using public and private key pair.
	- on your left panel of EC2 dashboard, click on 
	"network & security" category's "Key Pairs";
	- click on button "create key pair";
	- type your key pair name, for me, I put "EC2UbuntuLTSThreeT1Micro";
	- see a download file window, choose a secure folder to save your .pem file which will be your private key. For me, I get the file called "EC2UbuntuLTSThreeT1Micro.pem";

4. create EC2 instances
   * Each instance is like a seperate machine.
 	- on your left panel of EC2 dashboard,, click on 
	"instances" category's "instances";
	- click on button "launch instance";
	choose the system, find the 9th options--"Ubuntu Server 12.04 LTS (HVM) - ami-5fa4a236  64 bit";
	- click on "select";
	- keep the default setting, choose "next:configure instance details";
	- put "3" into "Number of instances";
	- click on "review and launch";
	- click on "launch";
	- choose the key pair "vivian_zhang_supstat";
	- check the acknowleage box;
	- click on "launch instance",
	you should see "Your instances are now launching";
    	scroll down to the bottom, click on "view instances",
    	click one each instance and rename them, such as "meetup1","meetup2","meetup3",
    	wait till you see the status checks is changed from "initializing" to "2/2 checks passed".

5. configure security group
	* Each group is like a firewall. The nodes of the same cluster belong to the same security group.

	* click on "create security groups",put name such as "meetup_security";

	* under "inbound" tab, cick on add rules;
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
   * know how to turn on and off the instances
	- select the instance, right click to choose "stop", you won't be able to use this instance and won't be charged;
   	- select the instance, right click to choose "start", you can use the instance and will be charged.
   	Next time, if you restart the instance, your "public DNS" will be different, but your "private DNS" will not changed.

Server configuration

0. You are required to use vi editor. The basic operations are:   

	```no-highlight
	 Cheet sheet for vi

    | what you want to do  | Key stokes       | 
    | ---------------------|:----------------:| 
    | insert/edit model    | i                | 
    | finish and save      | ESC->:->wq->Enter|
    | finish and not save  | ESC->q!->Enter   |
    | go to the end of line| ESC->o           |

1. generate your server rsa key for three instances

* find your meetup1's public DNS and get its RSA public key

    - open terminal 1, ssh to your server

      - before you run ssh, make sure you are at the location you can access your .pem file. I save .pem in .ssh folder, so I do "cd .ssh".
   	  - ssh  -i EC2UbuntuLTSThreeT1Micro.pem  ubuntu@ec2-54-209-171-193.compute-1.amazonaws.com
   	  - Are you sure you want to continue connecting (yes/no)? yes
      - generate your server key
      - ssh-keygen -t rsa 
      - do "Enter" three times
      - cd .ssh
      - vi id_rsa.pub
      - copy and paste it into a text file for future

* find your meetup2's public DNS and generate its RSA public key

    - open terminal 2, ssh to your server
    - two parts are different from meetup1
      - meetup2 has different public DNS address
      - copy and paste meetup2's id_ras.pub into the same file as second line

* find your meetup3's public DNS and generate its RSA public key

     - open terminal 3, ssh to your server
     - two parts are different  from meetup1
      - meetup3 has different public DNS address
      - copy and paste meetup3's id_ras.pub into the same file as third line

 * in the end, you should have a notepad which contains three rsa keys

 2. configure "authorized_keys" file for three instances

 * on meetup1 instance
  - cd .ssh
  - vi authorized_keys
  - go to the end of file by "ESC->o"
  - copy/paste three rsa keys to the end of file
    you should have one original keys and three new keys in the end
  - save and exit file by "ESC->wq!"

 * on meetup2 instance, do the same
 * on meetup3 instance, do the same

 4. configure "hosts" file for three instances

 * on meetup 1 instance
 	- sudo vi /etc/hosts
 	- find your private DNS from EC2 console
 	-- add three lines to the end of files, such as
 	172.31.43.177 meetup1
 	172.31.43.179 meetup2
 	172.31.43.178 meetup3
 * on meetup2 instance, do the same
 * on meetup3 instance, do the same

 5. test connections among cluster
  * from meetup1
  	- ping meetup2
  	- ping meetup3

  * from meetup2
  	- ping meetup1
  	- ping meetup3

  * from meetup3
  	- ping meetup1
  	- ping meetup2

  * ctrl +c to stop pinging
  

















