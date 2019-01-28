 === AWS Funtoo EC2 AMI building ===

 Refers to bootstrapping script here: https://paste.pound-python.org/show/7ibEdhjYKj2fdFe93UMd/
 Plus the 'aws ec2 register' command here: https://paste.pound-python.org/show/fJuhNWYBpRFet1XGZ1Uh/

 Bootstrapping manual steps:

 * Prerequisite: create an AWS account or use existing one
 * Startup an AWS EC2 Funtoo Linux instance (e.g. 64-bit, t2.micro, broadwell) 
 * Setup with 10GB harddrive (/dev/xvda), add a second hdd with 10GB (/dev/sdb which will get /dev/xvdb later)
 * SSH access is needed (must ensure the Security Group settings allow access to port 22, that is probably the default)
 * Launch the instance, create SSH keys if needed (this could be even your own keys, or stick with the one given by Amazon)
 * Connect to the instance via SSH (going to 'Services' > 'EC2', right-click on instance, choose connect and follow instructions)
 
 Annoyances:
 
  * When copying the SSH example from the UI the default AMI username is 'root' while the actual AMI username is 'ec2-user':
    - this leads to the problem that when copied and pasted in the terminal one will get asked for a password although
      the login will work with SSH keys only, just not for 'root' but 'ec2-user'
    - there seems no way to influence this, but maybe this can be done in the marketplace settings somewhere (default 'AMI username')
    - perhaps something like outputting text to console mentioning 'no login as 'root' possible. please use 'ec2-user'
      when the user just tries to execute the copied SSH example without changing username
      
 * Now that we are connected via SSH we just need to have a copy of the bootstrapping script (e.g. via scp before sshing)
 * sudo and execute the bootstrapping script
 * umount -R /mnt/gentoo (maybe 'sync' beforhand to ensure everything is written?)
 * shutdown the machine (or maybe just terminate it)
 * The secondary ebs volume (previously /dev/sdb) is now available (default is not to delete on instance termination)
   and there should a snapshot be made
 * Register snapshot with 'aws ec2 register-image' which create the AMI
 * Done :)
 
 Some more things I stumbeld upon:
 
 * Although 10 GB seems to be resonable small sized the actual minimum could possibly be 8 GB. The hdd will grow anyway
   afterwards when launching the instance, so there is no need to go with 10 GB initially IMHO.
 * there is already a package 'app-admin/amazon-ec2-init' which does the same as hardcoded in the bootstrapping script.
   but that is probably a bit outdated: maybe remove this package or update accordingly?
 * The Amazon UI System log is activated by adding 'console=ttyS0,115200n8' boot params ('console=tty0' boot param inside
   Amazon Linux instances seems to be there for instance screenshots, but that did not work and so that param seems
   not be needed). The output in the console shows some escape sequences as the serial console is actually color-less.
 * The Amazon Linux machines have some more boot params in grub, not sure if that make sense to explicit add these:
   'net.ifnames=0 biosdevname=0 nvme_core.io_timeout=4294967295'
 * I am not sure if the partitions are always properly aligned. AFAIK this could have an impact on performance. In any case,
   not too much experience with 'parted' yet...
