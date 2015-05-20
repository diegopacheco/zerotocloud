# Step 11 - Build and Bake AMI

In continuing with the layers that we'll use for our instances, we build a "Base AMI" that is used by every instance.
With this, we can install packages and configuration we believe all instances will need.
Ideally this leaves the baking of the actual application to just the application, keeping the bake quick and small.

# Install aminator

<a href="https://github.com/Netflix/aminator" target="_blank">Aminator</a> is a Python-based open sourced tool from Netflix to "Bake" Amazon images.
It can use system packages, like RPMs or DEBs, or via a system of plugins common configuration management tools, like Chef, Puppet or Ansible, to build a machine image.
It also supports various base environments. We'll be using Ubuntu but other RPM based OS's like CentOS are supported.

    sudo pip install git+https://github.com/Netflix/aminator.git@2.1.52-dev#egg=aminator

## IF You got a python error:  <BR>

ERROR:root:Error parsing  <BR>
Traceback (most recent call last):  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/core.py", line 109, in pbr  <BR>
    attrs = util.cfg_to_args(path)  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/util.py", line 243, in cfg_to_args  <BR>
    pbr.hooks.setup_hook(config)  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/hooks/__init__.py", line 25, in setup_hook  <BR>
    metadata_config.run()  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/hooks/base.py", line 27, in run  <BR>
    self.hook()  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/hooks/metadata.py", line 26, in hook  <BR>
    self.config['name'], self.config.get('version', None))  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/packaging.py", line 573, in get_version  <BR>
    version = _get_version_from_git(pre_version)  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/packaging.py", line 511, in _get_version_from_git  <BR>
    pre_version)  <BR>
  File "/usr/local/lib/python2.7/dist-packages/pbr/version.py", line 147, in from_pip_string  <BR>
    raise ValueError("Invalid version %r" % version_string)  <BR>
    
* Them you will need edit this file(setup.cfg):  <BR>
 $ git clone https://github.com/Netflix/aminator <BR> 
 $ cd aminator  <BR>
 $ sudo pip install -r requirements.txt
 $ cd /home/ubuntu/aminator <BR>
 $ vim setup.cfg <BR>
 <BR>
Search for the version and replace for: version = 2.1.65 IF you want use the latest release got here  <BR> https://github.com/Netflix/aminator/releases and check the latest number.   <BR>
 <BR>
$ sudo python setup.py install
    
# Build a Debian

To minimize complexity and to mimic how Netflix bakes, we're going to be building DEBs for baking. 
Once again we're using Gradle during this tutorial, but you can create your packages however you like. 
As stated above, you could use Chef or Puppet to instead. 

    cd ~/zerotocloud
    ./gradlew :baseami:buildDeb

In Gradle, we're using the ospackage plugin to build our packages. 
It lets us build RPM and DEB files with a single syntax declarative syntax, in Java.
E.g. this is what we're using to build the Base AMI:

    ospackage {
        requires('default-jdk')
        requires('tomcat7')

        from(file('root')) {
            into('/')
        }

        postInstall('rm -fr /var/lib/tomcat7/webapps/ROOT')
        postInstall("perl -p -i -e 's/port=\"8080\"/port=\"7001\"/gi' /var/lib/tomcat7/conf/server.xml")
    }

# Bake

We can now "bake".

    sudo aminate -e ec2_aptitude_linux -b ubuntu-foundation -n ubuntu-base-ami baseami/build/distributions/baseami_1.0.0_all.deb

It is run as root, via _sudo_, so that it can add and remove mount points.
The _-e_ argument tells Aminator what environment it is in.  In this case we tell it to use Aptitude.
The _-b_ argument is the name of the AMI to start with. This is the one we created in [Step 8](FoundationAMI.d).
The _-n_ argument says to use a "named" image, which is easier to keep track of. There will still be a ami-1234567 like name associated with the resulting AMI.
One note: Aminate will append a ‘-ebs’ suffix to the name. 
The baking process takes a few minutes. You can add _--debug_ to the command line to get more details of what it is doing.
