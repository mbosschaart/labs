*******************************
Lab 1 - Blueprint (MySQL)
*******************************

Connectivity Instructions:
**************************

+------------+--------------------------------------------------------+
| IP         |                                           Cluster IP   |
+------------+--------------------------------------------------------+
| Username   |                                           Cluster User |
+------------+--------------------------------------------------------+
| Password   |                                           Cluster Pass |
+------------+--------------------------------------------------------+

Lab Overview
************

Welcome to the Calm Hands-On-Lab - Simple Blueprint. What we’re going to
do here is make a basic blueprint:

.. figure:: http://s3.nutanixworkshops.com/calm/lab1/image1.png

This is a basic MySQL Deployment. In this lab we’ll start with a very
basic, single service.

Calm Glossary
*************

**Service**: One tier of a multiple tier application. This can be made
up of 1 more VMs (or existing machines) that all have the same config
and do the same thing

**Application (App):** A whole application with multiple parts that are
all working towards the same thing (for example, a Web Application might
be made up of an Apache Server, a MySQL database and a HAProxy Load
balancer. Alone each service doesn’t do much, but as a whole they do
what they’re supposed to).

**Macro:** A Calm construct that is evaluated and expanded before being
ran on the target machine. Macros and Variables are denoted in the
@@{[name]}@@ format in the scripts.

**Subtrate:** A Calm object used to encapsulate the VM(s) within a Blueprint 

Part 1: Accessing and Navigating Calm
**************************************

Getting Familiar with the Tools

1. Connect to https://<HPOC.PC:9440>

2. Login to Prism Central using the credentials specified above (use
   these credentials unless specified otherwise throughout this lab 

3. Click on the Apps tab across the top of Prism

Welcome to Calm! Upon accessing this page you will now notice a new ribbon along the left - this is used to navigate through Calm.

You are, by default, dropped into the Applications tab and can see all the instances of applications that have been launched from a blueprint.

Tabbed Navigation
=================

For now, let’s step through each tab:

.. figure:: http://s3.nutanixworkshops.com/calm/lab1/image2.png

Blueprint Editor Overview
=========================

Welcome to the Blueprint Editor! Let’s take a look at the interface.

.. figure:: http://s3.nutanixworkshops.com/calm/lab1/image4.png

There are 2 more buttons that are helpful to use while making a blueprint:

.. figure:: http://s3.nutanixworkshops.com/calm/lab1/image5.png


Part 2: Your Entry Level Blueprint
***********************************

This section provides the steps to create a simple service.

1. Navigate to the Blueprint (|image2|) tab.
2. Click on **Create Application Blueprint**.
3. Assign this Blueprint to the **Calm** project


Create Blueprint Workflow
=========================

.. note:: In general, the Blueprint creation flow goes:

1. Create Object in Application Overview or select existing object from the workspace or the Overview panel.
2. Configure the object in the configuration pane.
3. Repeat for each object.
4. Connect dependencies in the workspace.

Let’s get started by setting up the basics

Update the Blueprint Name to Calm_Workshop

Click on the **Credentials** button along the top of the Blueprint workspace. Update credentials as follows:

+-----------------------+---------------+
| **Name**              | CENTOS        |
+-----------------------+---------------+
| **Username**          | root          |
+-----------------------+---------------+
| **Secret**            | Password      |
+-----------------------+---------------+
| **Password**          | nutanix/4u    |
+-----------------------+---------------+
| **Use as Default**    | Checked       |
+-----------------------+---------------+

.. note:: Credentials are unique per Blueprint.

Setting Variables
=================

In this subsection we'll create some variables. It’s not necessary to do it at this point, however it will make things easier for the rest of the lab.

- Variables have 2 settings, **Secret** and **Runtime**. Normally variables are stored in plaintext and shown in the window here, the **Secret** setting changes that (perfect for passwords). **Runtime** specifies if this variable should be static (and only editable here) or should be able to be changed during the Launch Process.

- Variables can be referred to while configuring VMs using the **@@{variable\_name}@@** construct - Calm will evaluate and replace that string before sending it down to the VM.

.. figure:: http://s3.nutanixworkshops.com/calm/lab1/image8.png


**Setup the variables as specified in the table below:**

+----------------------+------------------------------------------------------+
| **Variable Name**    | **Value**                                            |
+----------------------+------------------------------------------------------+
| Mysql\_user          | root                                                 |
+----------------------+------------------------------------------------------+
| Mysql\_password      | nutanix/4u                                           |
+----------------------+------------------------------------------------------+
| Database\_name       | homestead                                            |
+----------------------+------------------------------------------------------+
| App\_git\_link       | https://github.com/ideadevice/quickstart-basic.git   |
+----------------------+------------------------------------------------------+


Adding A DB Service
===================

We'll now create the basic service.

- Click the + sign next to **Services** in the **Overview** pane.

- Notice that the **Configuration** pane has changed and there is now a box in the **Workspace.**

- Name your service **MYSQL** in the *Service Name* field.

- The *Substrate* section is the internal Calm name for this Service. Name this **MYSQLAHV**

- Make sure that the Cloud is set to **Nutanix** and the OS set to **Linux**

- Configure the VM as follows:

.. code-block:: bash

  VM Name .  : MYSQL
  Image .    : CentOS
  Disk Type .: DISK
  Device Bus : SCSI
  vCPU .     : 2
  Core/vCPU .: 1
  Memory     : 4 GB

- Scroll to the bottom and add the NIC **Primary** to the **MYSQL** VM.
- Configure the **Credentials** to use **CENTOS** created earlier.

Package Configuration
=====================

- Scroll to the top of the Service Panel and click **Package**.

- Name the install package **MYSQL_PACKAGE**

- Set the install script to **shell** and select the credential **CENTOS** created earlier.

- Copy the following script into the *script* field of the **install** window:

.. code-block:: bash

   #!/bin/bash
   set -ex

   yum install -y "http://repo.mysql.com/mysql-community-release-el7.rpm"
   yum update -y
   yum install -y mysql-community-server.x86_64

   /bin/systemctl start mysqld

   #Mysql secure installation
   mysql -u root<<-EOF

   #UPDATE mysql.user SET Password=PASSWORD('@@{Mysql_password}@@') WHERE User='@@{Mysql_user}@@';
   DELETE FROM mysql.user WHERE User='@@{Mysql_user}@@' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
   DELETE FROM mysql.user WHERE User='';
   DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%';

   FLUSH PRIVILEGES;
   EOF

   sudo yum install firewalld -y
   sudo service firewalld start
   sudo firewall-cmd --add-service=mysql --permanent
   sudo firewall-cmd --reload

   #mysql -u @@{Mysql_user}@@ -p@@{Mysql_password}@@ <<-EOF
   mysql -u @@{Mysql_user}@@ <<-EOF
   CREATE DATABASE @@{Database_name}@@;
   GRANT ALL PRIVILEGES ON homestead.* TO '@@{Database_name}@@'@'%' identified by 'secret';

   FLUSH PRIVILEGES;
   EOF


- Looking at this script, we see that we’re using the variables we set before and doing basic mySQL configuration. This can be customized for whatever unique need you have.

- Since we don’t need anything special ran when uninstalling, we will just add a very basic script to the uninstall. This can be useful for cleanup (for example, releasing DNS names or cleaning up AD), but we won’t use it here.

- Set the uninstall script to **shell** and select the credential **CENTOS** created earlier.

- Add the following to the *script* field in the **uninstall** window:

.. code-block:: bash

   #!/bin/bash
   echo "Goodbye!"

- After completing the configuration, click the **Save** button. If any errors come up, go back and review the configuration to ensure that all fields have been filled.

Part 3: Launching the Blueprint
*******************************

Now that the blueprint has been created and saved, you can launch it!

- Click on the **Launch** button in the top right of the blueprint. This will bring up the the launch window.
- Give this instance a unique name **Calm_Workshop_MYSQL_App_1**.

.. note:: Every launch performed requires a name change, making each launch unique - this can be done by incrementing the suffix in the name.

- This will now bring you to the **Instance** page. The bar across the top allows you to see various information about the application instance:

.. figure:: http://s3.nutanixworkshops.com/calm/lab1/image25.png


Summary
*******

You've successfully created a blueprint, and launched it as an application.

.. |image0| image:: lab1/media/image1.png
.. |image1| image:: lab1/media/image2.png
.. |image2| image:: http://s3.nutanixworkshops.com/calm/lab1/image3.png
.. |image3| image:: lab1/media/image4.png
.. |image4| image:: lab1/media/image5.png
.. |image5| image:: http://s3.nutanixworkshops.com/calm/lab1/image6.png
.. |image6| image:: lab1/media/image7.png
.. |image7| image:: lab1/media/image8.png
.. |image10| image:: lab1/media/image11.png
.. |image11| image:: lab1/media/image12.png
.. |image12| image:: lab1/media/image13.png
.. |image13| image:: lab1/media/image14.png
.. |image14| image:: lab1/media/image15.png
.. |image15| image:: lab1/media/image16.png
