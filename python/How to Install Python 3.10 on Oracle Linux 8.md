# How to Install Python 3.10 on Oracle Linux 8 
Python is a free, open-source, powerful, and widely used programming language. It is an object-oriented application used in automation, scripting, data analysis, machine learning, back-end development, handling big data, and performing complex mathematics. It is the first choice for both beginners and experienced developers due to its versatile features. At the time of writing this article, Python 3.10.4 is the latest version of Python.

In this post, we will show you how to install Python 3.10 on Oracle Linux 8.

Prerequisites
-------------

*   A server running Oracle Linux 8 on the Atlantic.Net Cloud Platform
*   A root password configured on your server

Step 1 – Create Atlantic.Net Cloud Server
-----------------------------------------

First, log in to your [Atlantic.Net Cloud Server](https://cloud.atlantic.net/?page=userlogin). Create a new [server](https://www.atlantic.net/cloud-hosting/how-to-create-new-atlantic-net-cloud-server/), choosing Oracle Linux 8 as the operating system with at least 2GB RAM. Connect to your Cloud Server via SSH and log in using the credentials highlighted at the top of the page.

Once you are logged in to your server, run the following command to update your base system with the latest available packages.

dnf update -y

Step 2 – Install Required Dependencies
--------------------------------------

The latest version of Python is not included in the Oracle Linux 8 default repo, so you will need to compile it from the source.

To compile Python from the source, you will need to install some dependencies on your system. You can install all of them by running the following command:

dnf install curl gcc openssl-devel bzip2-devel libffi-devel zlib-devel wget make -y

Once all the dependencies are installed, you can proceed to the next step.

Step 3 – Install Python 3.10.4 on Oracle Linux 8
------------------------------------------------

Next, visit the Python official download page and download the latest version of Python using the following command:

wget https://www.python.org/ftp/python/3.10.4/Python-3.10.4.tgz

Once the download is completed, extract the downloaded file using the following command:

tar -xf Python-3.10.4.tgz

Next, change the directory to the extracted directory and configure Python using the following command:

cd Python-3.10.4
./configure --enable-optimizations

Next, start the build process using the following command:

make -j 2
nproc

Finally, install Python 3.10 by running the following command:

make altinstall

After the successful installation, verify the Python installation using the following command:

python3.10 --version

You will get the following output:

Python 3.10.4

Step 4 – Create a Virtual Environment in Python
-----------------------------------------------

Python provides a venv module that helps developers to create a virtual environment and deploy applications easily in an isolated environment.

To create a virtual environment named python-env, run the following command:

python3.10 -m venv python-env

Next, activate the virtual environment using the following command:

source python-env/bin/activate

You will get the following shell:

(python-env) \[root@oraclelinux8 ~\]#

Now, you can use the PIP package manager to install any package and dependencies inside your virtual environment.

For example, run the following command to install apache-airflow:

pip3.10 install apache-airflow

If you want to remove this package, run the command below:

pip3.10 uninstall apache-airflow

To exit from the Python virtual environment, run the following command:

deactivate

Conclusion
----------

In this guide, we explained how to install Python 3.10.4 on Oracle Linux 8. You can now install Python in the development environment and start developing your first application using the Python programming language. Try it on [dedicated hosting](https://www.atlantic.net/dedicated-server-hosting/) from Atlantic.Net!