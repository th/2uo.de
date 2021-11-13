# Setting up a free server in Oracle Cloud

Oracle Cloud offers a free tier that includes a beefy ARM64 virtual server. with lots of RAM.
If you forgo the two “micro instances” you can have an Ampere (ARM64) virtuals server with 4 cores and 24 GB of RAM for free.

## Creating the instance

After you've signed up for Oracle Cloud you need to create your server instance. The dashboard is huge, confusing and full of enterprise features.

Start at the “Compute” section and choose “Instances”.

![Compute section](setting-up-a-free-server-in-oracle-cloud-menu-compute.png)

Then click “Create instance”.

### Name

![Create instance](setting-up-a-free-server-in-oracle-cloud-create-instance-1.png)

Give a name to the instance and leave the availability domain at whatever it is. You'll need to use the same availability domain for the block storage later, so remember it.

### Shape and image

Edit “Image and shape”. You want an Ampere instance, so set the check box next to “VM.Standard.A1.Flex”. Then enter 4 OCPUs and 24 GB of RAM.

You could probably go higher, but then you'll pay after the trial period. So keep it to the 4 OCPUs and 24 GB of RAM setting, which is within the free tier.

The image doesn't matter much, since I will replace it later; Ubuntu or Oracle Linux are both fine.

### Networking

![Create instance](setting-up-a-free-server-in-oracle-cloud-create-instance-2.png)

Under “Networking” you can leave everything as-is, but I like to set “Do not assign a private DNS record”.

### SSH

![Create instance](setting-up-a-free-server-in-oracle-cloud-create-instance-3.png)

Now paste a prepared SSH **public key** into the text field.

![Create instance](setting-up-a-free-server-in-oracle-cloud-create-instance-4.png)

#### Creating a SSH key

If you don't already have one ready, install PuTTY and start PuTTYgen.

![PuTTYgen](setting-up-a-free-server-in-oracle-puttygen-1.png)

Click “Generate”, move the mouse a bit, and then use the highlighted text as your SSH key in the Oracle Cloud dashboard.
You should also save it in a text file somewhere.

![PuTTYgen](setting-up-a-free-server-in-oracle-puttygen-2.png)

PuTTY saves the private key in its own format, that other tools won't be able to handle.
So select “Conversions” – “Export OpenSSH key” and save the private key in OpenSSH format.

**Don't lose that private key!**

If you later need the public key again, but didn't save it, you can always load the private key (from PuTTY's own file format)
and see the public key again in the top field.

![PuTTYgen](setting-up-a-free-server-in-oracle-puttygen-3.png)

### Finish

Click “Create” and after a short waiting time your instance is provisioned and ready.

![Create instance](setting-up-a-free-server-in-oracle-cloud-create-instance-5.png)

## Attaching block storage

Your instance has got a boot volume (think: virtual hard disk) of 50 GB. You can change that value, but I keep it at the default value.

The free tier allows you 200 GB, so let's use the missing 150GB!

Go to the “Storage” section and select “Block Volumes”.

![Storage](setting-up-a-free-server-in-oracle-cloud-menu-block-volume.png)

Give the new block volume a name. Make sure the availability domain is the same as when you created your instance.

![Create a block volume](setting-up-a-free-server-in-oracle-cloud-create-block-volume-1.png)

Select “Custom” and change the size from the default value 50 to 150. Everything else is fine.

![Create a block volume](setting-up-a-free-server-in-oracle-cloud-create-block-volume-2.png)

Now go back to your instance. In the lower left menu click “Attached block volumes” and then “Attach block volume”.

![Attach a block volume](setting-up-a-free-server-in-oracle-cloud-attach-block-volume-1.png)

Select your newly created block volume and click “Attach”.

![Attach a block volume](setting-up-a-free-server-in-oracle-cloud-attach-block-volume-2.png)

## Configure networking
 
 On to “Networking”!  Oracle has a default access list that blocks some traffic that I want to get rid of.
 
 If you've chosen Ubuntu as your image, don't you'll need to follow the instruction in [Firewall in Oracle Cloud](firewall-in-oracle-cloud.md) later.
 
 ![Networking](setting-up-a-free-server-in-oracle-cloud-menu-networking.png)
 
Select the vcn (that was created automatically).
 
 ![Networking](setting-up-a-free-server-in-oracle-cloud-networking-1.png)
 
And the subnet within.

![Networking](setting-up-a-free-server-in-oracle-cloud-networking-2.png)

Within the subnet is a “Default Security List”. Click on it and “Add Ingree Rule”

![Networking](setting-up-a-free-server-in-oracle-cloud-networking-3.png)

Add an ingress rule that allows everything: Source CIDR is 0.0.0.0/0, protocol is “TCP” leave port range free.

![Networking](setting-up-a-free-server-in-oracle-cloud-networking-4.png)
 
## Done (for now)

You can login to your instance using your SSH key now. For Ubuntu you login as root, for Oracle Linux you'll use the user name opc.

## Debian

But I want Debian installed. You had no option to choose an ISO image anywhere when you created the instance, so now we resort to trickery.

