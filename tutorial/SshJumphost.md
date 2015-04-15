# Step 9 - Setup Jumphost

The jumphost up to this point is running a stock copy of Ubuntu, so we're going to jump onto it and configure it. 
The first half will be done locally and the second half (and most of the rest of the tutorial) will be done on the instance.
We've chosen to do as much as possible on the command line, to help ensure that the steps can be followed exactly. Commands are expected to be run exactly as provided and are formatted like `this`.
The commands we're using locally are _ssh_ and _scp_, which will be available on a Mac or on a Linux box by default.
 
### PuTTY

<a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html" target="_blank">On Windows, Amazon recommends using PuTTY</a>.
Please follow their instructions when it pertains to connecting to your instance below.

Make sure you genereate your private KEY using puttygen, follow instructions here: https://www.youtube.com/watch?v=mJaHARCfcA0

## 1. Terminal
 
Start at a terminal on your laptop, then get closer to that pem file we downloaded earlier:

For UNIX, we also have to tighen down the permissions of the pem file for ssh to let us use it:

    cd ~/Downloads
    chmod 0600 zerotocloud.pem
    
For Windows, use the instructions above to convert your .pem to a .ppk using PuTTYgen and save the file to My Documents. Use a cmd.exe terminal for further commands.

    cd "My Documents"

The key parts will be the key and the host name prefixed by "ubuntu@". ![](images/Putty-host.png) ![](images/Putty-key.png)

## 2. Save Jumphost

The jumphost created in [Step 4](Jumphost.md) will be referenced a few times, so for convenience let's save it's address.

For UNIX:

    export JUMPHOST=ec2-54-191-135-15.us-west-2.compute.amazonaws.com
    
For windows:

    set JUMPHOST=ec2-54-191-135-15.us-west-2.compute.amazonaws.com
    
## 3. Upload files to jumphost

When prompted about the RSA fingerprint, type "yes".

For UNIX:

    scp -i zerotocloud.pem credentials.csv ubuntu@$JUMPHOST:credentials.csv
    scp -i zerotocloud.pem zerotocloud.pem ubuntu@$JUMPHOST:zerotocloud.pem
    
For Windows:

    "C:\Program Files\PuTTY\pscp.exe" -i zerotocloud.ppk credentials.csv ubuntu@%JUMPHOST%:credentials.csv
    "C:\Program Files\PuTTY\pscp.exe" -i zerotocloud.ppk zerotocloud.pem ubuntu@%JUMPHOST%:zerotocloud.pem

## 4. Log onto jumphost

From this point on, we'll be on the jumphost and not on your laptop. We're adding a _-L_ argument here for a later steps.
 
For UNIX:

    ssh -i zerotocloud.pem -L 8080:localhost:8080 ubuntu@$JUMPHOST

For Windows, use the instructions above, with the one additional change of port forwarding. ![](images/Putty-forward.png)

## 5. Update jumphost

These commands are executed on the jumphost, to add some commands that we're going to need. 
The second command will prompt you to continue, hit Return. Both commands will take a few minutes to run.

    sudo apt-get update
    sudo apt-get install python-pip python-dev git default-jdk


# Troubleshooting

## Mosh (advanced)

If the network connection is sub-par, an alternative to _ssh_ that is more tolerant to network issues is [mosh](mosh.mit.edu). 
From the jumphost install the server-component:

    sudo apt-get install python-software-properties
    sudo add-apt-repository ppa:keithw/mosh
    sudo apt-get update
    sudo apt-get install mosh
    
From your local laptop follow the instructions to install the client component on <a href="http://mosh.mit.edu/#getting" target="_blank">their website</a>. 
Even though they have a version for Chrome, it doesn't support provide our own credential pem file. 
On a Mac, it will warn about an "unidentified developer", to get around this Ctrl click on it, then select open.

Mosh doesn't support port forwarding, so in a later step when we start Asgard on the jumphost we won't be able to access it. 
To work around this, ths jumphost's Security Group, with a name like launch-wizard-1, will need to be altered. 
We also need UDP access on port 60001. 

1. From the console, find your instance. And click on the security group in the lower window.
2. While your security group is selected, change to the Inbound tab.
3. Click "Edit"
4. In the dialog, add a Custom UDP rule for 60001 from Anywhere.
5. Add a Custom TCP rule for 8080 from Anywhere.
6. Close.

Once that's all done you can run this command:

    mosh --ssh="~/bin/ssh -i zerotocloud.pem" ubuntu@$JUMPHOST
    
Caveat emptor, we are opening up port that could have vulnerabilities.
