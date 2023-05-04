Download Link: https://assignmentchef.com/product/solved-csc555-mining-big-data-project-phase-1
<br>
In this part of the project (which will also serve as our take-home midterm), you will 1) Set up a 4-node cluster and 2) perform data warehousing and transformation queries using Hive, Pig and Hadoop streaming. The modified Hive-style schema is at:

http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/SSBM_schema_hive.sql

It is based on SSBM benchmark (derived from industry standard TPCH benchmark). The data is at Scale1, or the smallest unit – lineorder is the largest table at about 0.6GB. You can use wget to download the following links. Keep in mind that data is |-separated (not csv).

<a href="http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/dwdate.tbl">http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/dwdate.tbl</a>

<a href="http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/lineorder.tbl">http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/lineorder.tbl</a>

<a href="http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/part.tbl">http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/part.tbl</a>

<a href="http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/supplier.tbl">http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/supplier.tbl</a>

<a href="http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/customer.tbl">http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/customer.tbl</a>

Please be sure to <u>submit all code</u> (pig, python and SQL).

<h1>Part 1: Multi-node cluster</h1>




<ul>

 <li>Your first step is to setup a multi-node cluster and re-run a simple wordcount. For this part, you will create a 4-node cluster (with a total of 1 master + 3 worker nodes). Include your master node in the “slaves” file, to make sure all 4 nodes are working.</li>

</ul>

You need to perform the following steps:




<ol>

 <li>Create a new node of a medium size (you can always switch the size of the node). It is possible, but I do not recommend trying to reconfigure your existing Hadoop into this new cluster (it is much easier to make 4 new nodes for a total of 5 in your AWS account).</li>

</ol>




<ol>

 <li><strong>When creating a node I recommend changing the default 8G hard drive to 30G on all nodes.</strong></li>

</ol>




<ol>

 <li>Change your security group setting to open firewall access. We need to open the ports in two different ways. We will open port 50070 for the web interface in order to be able to see the cluster status in a browser. We will also set 0-64000 range opening up all ports. However, we will ensure that the ports are open only <strong>within</strong> the cluster and not to the world.</li>

</ol>




In order to make changes, you need to do the following. Access the cluster security group (launch-wizard-xx). Right click on the security group and choose Edit inbound rules




Note that the first line below is opening port 50070. The second line below is the default (port 22 is required for regular SSH connections). The third line opens all ports but ONLY for the same security group (assuming that all of your nodes in the cluster share the same security group – that will happen automatically if you use the “create more like this” option when creating instances as specified in part 1-c below). We had some issues with machines being hacked and disabled without that last limitation, so make sure you include it.

<ol>

 <li>Right click on the Master node and choose “create more like this” to create 3 more nodes with same settings. If you configure the network settings on master first, security group information will be copied.</li>

</ol>

NOTE: Hard drive size will not be copied and default to 8G unless you change it.




<ol start="2">

 <li>Connect to the master and set up Hadoop similarly to what you did previously. Do not attempt to repeat these steps on workers yet – you will only need to set up Hadoop <u>once</u>.</li>

</ol>




<ol>

 <li>Configure core-site.xml, adding the <strong>PrivateIP</strong> (do not use public IP) of the master.</li>

</ol>




<ol start="2">

 <li>Configure hdfs-site and set replication factor to 2.</li>

</ol>




<ol start="2">

 <li>cp hadoop-2.6.4/etc/hadoop/mapred-site.xml.template hadoop-2.6.4/etc/hadoop/mapred-site.xml and then configure mapred-site.xml</li>

</ol>




<ol>

 <li>Configure yarn-site.xml (once again, use PrivateIP of the master)</li>

</ol>

Finally, edit the slaves file and list your 4 nodes (master and 3 workers) using Private IPs

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="0a6f6938277f796f784a637a273b3d3827393b273d27383a3b">[email protected]</a> ~]$ cat hadoop-2.6.4/etc/hadoop/slaves

172.31.7.201

172.31.5.246

…




Make sure that you use <u>private IP</u> (private DNS is also ok) for your configuration files (such as conf/masters and conf/slaves or the other 3 config files). The advantage of the Private IP is that it does not change after your instance is stopped (if you use the Public IP, the cluster would need to be reconfigured every time it is stopped). The downside of the Private IP is that it is only meaningful within the Amazon EC2 network. So all nodes in EC2 can talk to each other using Private IP, but you <u>cannot</u> connect to your instance from the outside (e.g., from your laptop) because Private IP has no meaning for your laptop (since your laptop is not part of the Amazon EC2 network).




Now, we will pack up and move Hadoop to the workers. All you need to do is to generate and then copy the public key to the worker nodes to achieve passwordless access across your cluster.




<ol>

 <li>Run ssh-keygen -t rsa (and enter empty values for the passphrase) on the <u>master</u> That will generate .ssh/id_rsa and .ssh/id_rsa.pub (private and public key). You now need to manually copy the .ssh/id_rsa.pub and append it to ~/.ssh/authorized_keys <strong><u>on each worker. </u></strong></li>

</ol>

Keep in mind that this is a single-line public key and accidentally introducing a line break would cause a mismatch.

Note that the example below is NOT the master, but one of the workers (ip-172-31-5-246). The first public key is the .pem Amazon half and the 2<sup>nd</sup> public key is the master’s public key copied in as one line.




You can add the public key of the master to the master by running this command:

cat ~/.ssh/id_rsa.pub &gt;&gt; ~/.ssh/authorized_keys

Make sure that you can ssh to all of the nodes <u>from the master node</u> (by running ssh 54.186.221.92, where the IP address is your worker node) from the master and ensuring that you were able to login.  You can exit after successful ssh connection by typing exit (the command prompt will tell you which machine you are connected to, e.g., <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="a6c3c5948bd3d5c3d4e6cfd68b9791948b95978b95918b979795">[email protected]</a>).  Here’s me ssh-ing from master to worker.




Once you have verified that you can ssh from the master node to every cluster member including the master itself (ssh localhost), you are going to return to the master node (exit until your prompt shows the IP address of the master node) and pack the contents of the hadoop directory there. Make sure your Hadoop installation is configured correctly (because from now on, you will have 4 copies of the Hadoop directory and all changes need to be applied in 4 places).




cd (go to root home directory, i.e. /home/ec2-user/)

(pack up the entire Hadoop directory into a single file for transfer. You can optionally compress the file with gzip)

tar cvf myHadoop.tar hadoop-2.6.4

ls -al myHadoop.tar (to verify that the .tar file had been created)




Now, you need to copy the myHadoop.tar file to every non-master node in the cluster. If you had successfully setup public-private key access in the previous step, this command (for <u>each</u> worker node) will do that:




(copies the myHadoop.tar file from the current node to a remote node into a file called myHadoopWorker.tar. Don’t forget to replace the IP address with that your worker nodes. By the way, since you are on the Amazon EC2 network, either Public or Private IP will work just fine.)

scp myHadoop.tar <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="1e7b7d2c336b6d7b6c5e2b2a302f262930282d302f2627">[email protected]</a>:/home/ec2-user/myHadoopWorker.tar




Once the tar file containing your Hadoop installation from master node has been copied to each worker node, you need to login to each non-master node and unpack the .tar file.




Run the following command (on each worker node, not on the master) to untar the hadoop file. We are purposely using a different tar archive name (i.e., myHadoopWorker.tar), so if you get “file not found” error, that means you are running this command on the master node or have not yet successfully copied myHadoopWorker.tar file to the worker.




tar xvf myHadoopWorker.tar




Once you are done, run this on the master (nothing needs to be done on the workers to format the cluster unless you are re-formatting, in which case you’ll need to delete the dfs directory).

hadoop namenode -format




Once you have successfully completed the previous steps, you should can start and use your new cluster by going to the master node and running the start-dfs.sh and start-yarn.sh scripts (you <u>do not</u> need to explicitly start anything on worker nodes – the master will do that for you).




You should verify that the cluster is running by pointing your browser to the link below.




http://[insert-the-public-ip-of-master]:50070/




Make sure that the cluster is operational (you can see the 4 nodes under Datanodes tab).




<u>Submit a screenshot of your cluster status view</u>.




Repeat the steps for wordcount using bioproject.xml from Assignment 1 and submit screenshots of running it.

<u> </u>

<u>Submit a short paragraph with a discussion about how the results compare (faster? slower? How much faster/slower? Due to what?) </u>




<h1>Part 2: Hive</h1>

Run the following three (1.2, 1.3 and 2.1) queries in Hive and record the time they take to execute:                                                        <a href="http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/SSBM_queries.sql">http://rasinsrv07.cstcis.cti.depaul.edu/CSC555/SSBM1/SSBM_queries.sql</a>

Perform the following transform operation using SELECT TRANSFORM on the customer table by creating a new table:

For the c_address column, shorten it to 8 characters (i.e., if the value is longer, remove extra characters, but otherwise keep it as-is). For c_city, add a space and a # to indicate the digit at the end (e.g., UNITED KI2 =&gt; UNITED KI #2, or INDONESIA4 =&gt; INDONESIA #4). Make sure to modify the columns of the target table accordingly (since you are introducing longer columns).

<h1></h1>

<h1>Part 3: Pig</h1>

Convert and load the data into Pig, <u>implementing only queries 0.1, 0.2, 0.3</u>. <u>Do not implement all queries</u>.

Check disk storage space in HDFS, if your disk usage is over 90% Pig may hang without an error or a warning.

One easy way to time Pig is as follows: put your sequence of pig commands into a text file and then run, from command line in pig directory (e.g., [<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="600503524d151305122009104d5157524d53514d564d5359">[email protected]</a> pig-0.15.0]$), bin/pig -f pig_script.pig (which will inform you how long the pig script took to run).

<h1></h1>

<h1>Part 4: Hadoop Streaming</h1>

Implement queries <strong>0.2 and 0.3</strong> using Hadoop streaming with python.

NOTE: You may implement this part in Java if you prefer.

<u>Submit a single document containing your written answers.  Be sure that this document contains your name and “CSC 555 Project Phase 1” at the top.</u>