# Heat Templates
Heat is a service to orchestrate multiple composite cloud applications using templates.

### Heat template orchestration guide
https://docs.openstack.org/heat/rocky/template_guide/hot_guide.html

# Heat-stack-workshop
This repository contains templates for use with [OpenStack Heat](https://wiki.openstack.org/wiki/Heat) to deploy openstack resources.

# Pre-requisites

### OpenStack CLI tools

You need to install the following packages: 
python3-openstackclient 
python3-novaclient 
python3-heatclient 
python3-neutronclient 
python-glanceclient

This will provide commands such as `openstack`, `nova`, `heat`, etc.

### An OpenStack account

You need to have access to an OpenStack environment and project which allows
you to:

* Create/delete virtual machines and networks
* Create/run/delete Heat stacks
* Assign floating IP addresses

You will need to have an OPENRC file.  Here is an example, using Keystone V3
auth:
```
export LC_ALL="en_US.UTF-8"
export OS_AUTH_TYPE=password
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=<project_name>
export OS_USERNAME=my-username
export OS_PASSWORD=my-password
export OS_AUTH_URL=https://<URL>:30500/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export OS_INTERFACE=public
```
### Heat stack deployment

  1. Grab repo from github:
     ```https://github.com/hemantsonawane95/heat-stack-workshop.git```
  2. Now do the changes in the confiugration files accordingly.
     
     The following paramerters must be change:
     * 'flavor' : Put a valid flavor name from openstack
     * 'image'  : A valid image name or ID from openstack
     * 'ssh_key' : ssh key name imported in openstack
     * 'heat_volume_size' : specify the external volume size which needs to be attached to the server
     * 'heat_floating_ip_net' : specify valid floating ip network name
  3. source openrc file using command `source openrc`

Note: The template must be in yaml format and be saved with the .yaml file extension. You should also make sure that you use the correct indentation. 

### Task-1 : Volume creation in openstack using heat-template

The ultimate goal of this task is to create volume in openstack env using heat-template. Use `heat-volume-stack.yaml` template to create volume. 
By running openstack heat command `openstack heat create` will create 10GB size volume in openstack env. It can be verified via openstack cli or in openstack horizon. 

For resizing volume change the default size in `heat-volume-stack.yaml` and then running stack update will resize the volume to updated size.


### Task-2 : Create full resources in openstack 

This task is all about creating resources needed to run a server in openstack env using heat-template. Use `heat-resources-stack.yaml` template to create resources.

This template will create the following and connect them up:
  
  * Instance
  * router
  * port 
  * static ip 
  * floating ip 
  * volume 
  * Instance 
  
**Create an Instance:**

Use the `OS::Nova::Server` resource to create a Compute instance. The `flavor` property is the only mandatory one, but you need to define a boot source using one of the `image` or `block_device_mapping` properties.
You also need to define the networks property to indicate to which networks your instance must connect if multiple networks are available in your tenant.

**Connect an instance to a network:**

Use the networks property of an `OS::Nova::Server` resource to define which networks an instance should connect to. Define each network as a YAML map, containing one of the following keys:

**port:**

The ID of an existing Networking port. You usually create this port in the same template using an `OS::Neutron::Port` resource. You will be able to associate a floating IP to this port, and the port to your Compute instance.

**network:**

The name or ID of an existing network. You don’t need to create an `OS::Neutron::Port` resource if you use this property. But you will not be able to use neutron floating IP association for this instance because there will be no specified port for server.

**Create and associate a floating IP to an instance:**

You can use two sets of resources to create and associate floating IPs to instances.
 
**1. OS::Nova resources:**
    Use the `OS::Nova::FloatingIP` resource to create a floating IP, and the `OS::Nova::FloatingIPAssociation` resource to associate the floating IP to an instance.
 
**2. OS::Neutron resources:**
    Use the `OS::Neutron::FloatingIP` resource to create a floating IP, and the `OS::Neutron::FloatingIPAssociation` resource to associate the floating IP to a port
    we have used this set to create floating ip and attached to instance in this workshop.

**Enable remote access to an instance:**
  The `key_name` attribute of the `OS::Nova::Server` resource defines the key pair to use to enable SSH remote access
  We can also specify `key_name` as parameter same as in `heat-resources-stack.yaml` template or key can be created.

**Create a network and a subnet:**
  Use the `OS::Neutron::Net` resource to create a network, and the `OS::Neutron::Subnet` resource to provide a subnet for this network

**Create and manage a router:** 
  Use the `OS::Neutron::Router` resource to create a router. You can define its gateway with the `external_gateway_info` property
  
**Create a volume:**
  Use the `OS::Cinder::Volume` resource to create a new Block Storage volume. Also we can create it as parameter and use it in resources.

**Attach a volume to an instance:**
  Use the OS::Cinder::VolumeAttachment resource to attach a volume to an instance.

For more details about heat-template orchestration guide please check official documentation metioned above.
