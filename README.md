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
    
### Task-1 : Volume creation in openstack using heat-template

Lets create single volume in openstack env by running `heat-volume-stack.yaml` file
    
        openstack stack create -t heat-volume-stack.yaml <stack_name>
    
        +---------------------+--------------------------------------+
        | Field               | Value                                |
        +---------------------+--------------------------------------+
        | id                  | 4cf6a524-eb8b-4d74-b450-8a8d16bceb0d |
        | stack_name          | volume-stack                         |
        | description         | Heat stack workshop                  |
        | creation_time       | 2021-11-26T07:33:54Z                 |
        | updated_time        | None                                 |
        | stack_status        | CREATE_IN_PROGRESS                   |
        | stack_status_reason | Stack CREATE started                 |
        +---------------------+--------------------------------------+
        
This will create a single volume of 10Gb size as follows: 

        openstack volume list
        
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | ID                                   | Name                                | Status    | Size | Attached to |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | cb7f8f2e-d5bd-42e4-bcb4-b62f63dcc8a6 | volume-stack-volume-01-26jz4xci46ia | available |   10 |             |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
  
To resize volume change the `default` size in heat-volume-stack.yaml file to required value:

     parameters:
       heat_volume_size:
         type: number
         label: Volume Size (GB)
         description: External Volume size in GB
         default: 20
    
Run `openstack stack update -t heat-volume-stack.yaml <stack-name>`

        +---------------------+--------------------------------------+
        | Field               | Value                                |
        +---------------------+--------------------------------------+
        | id                  | 4cf6a524-eb8b-4d74-b450-8a8d16bceb0d |
        | stack_name          | volume-stack                         |
        | description         | Heat stack workshop                  |
        | creation_time       | 2021-11-26T07:33:54Z                 |
        | updated_time        | 2021-11-26T07:34:24Z                 |
        | stack_status        | UPDATE_IN_PROGRESS                   |
        | stack_status_reason | Stack UPDATE started                 |
        +---------------------+--------------------------------------+
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | ID                                   | Name                                | Status    | Size | Attached to |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | cb7f8f2e-d5bd-42e4-bcb4-b62f63dcc8a6 | volume-stack-volume-01-26jz4xci46ia | available |   20 |             |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+

### Task-2 : Create full resources in openstack 

Create server with resources such as network, volume, floating ip, ports etc. 
       
Similarly, as we created volume using heat template this script will create all the necessary resources along with server.

        openstack stack create -t heat-resources-stack.yaml <stack_name>
     
        +---------------------+--------------------------------------+                                                                                
        | Field               | Value                                |                                                                                             
        +---------------------+--------------------------------------+                                                         
        | id                  | 9e90e65e-d2a8-428a-81ce-0d0223333273 |     
        | stack_name          | heat-stack                           |        
        | description         | Heat stack workshop                  |   
        | creation_time       | 2021-11-26T10:12:35Z                 |    
        | updated_time        | None                                 |
        | stack_status        | CREATE_IN_PROGRESS                   |            
        | stack_status_reason | Stack CREATE started                 |                                           
        +---------------------+--------------------------------------+    

You will see that all your resources got created. You can verify cretead resources via commandline or via openstack horizon:
         
        openstack server list 

        +--------------------------------------+-------------+--------+--------------------------------------+------------------------------+-----------+
        | ID                                   | Name        | Status | Networks                             | Image                        | Flavor    |
        +--------------------------------------+-------------+--------+--------------------------------------+------------------------------+-----------+
        | 0d2cc734-8de5-4f0d-b902-61c9cf5f4611 | heat_server | ACTIVE | heat_network=10.1.1.96, 185.22.97.37 | bionic-server-cloudimg-amd64 | m1.medium |
        +--------------------------------------+-------------+--------+--------------------------------------+------------------------------+-----------+
        
        openstack network list

        +--------------------------------------+--------------+--------------------------------------+
        | ID                                   | Name         | Subnets                              |
        +--------------------------------------+--------------+--------------------------------------+ 
        | 30577cc5-2e52-4861-b52d-33dde4ac772e | heat_network | 0037fdd9-d617-420a-ae0d-3b83b3129f0a |  <---- heat-template created network
        | 4acd59cc-7f03-4939-8dc4-841ec7528ca6 | public2      | c1a71a8b-ddcd-4345-ae80-a0bf75a9d5d0 |  <---- public network (floating ip pool)
        +--------------------------------------+--------------+--------------------------------------+

        openstack subnet list
       +--------------------------------------+---------------------------------------------+--------------------------------------+----------------+
       | ID                                   | Name                                        | Network                              | Subnet         |
       +--------------------------------------+---------------------------------------------+--------------------------------------+----------------+
       | 0037fdd9-d617-420a-ae0d-3b83b3129f0a | heat-stack-heat_network_subnet-e3vtk3lpuiat | 30577cc5-2e52-4861-b52d-33dde4ac772e | 10.1.1.0/24    |
       +--------------------------------------+---------------------------------------------+--------------------------------------+----------------+
        
        openstack port list
        
       +--------------------------------------+------------------------------------------+-------------------+------------------------------------------+--------+
       | ID                                   | Name                                     | MAC Address       | Fixed IP Addresses                       | Status |
       +-------------------------------+------------------------------------------+-------------------+-------------------------------------------------+--------+
       | ac90e42c-7548-4915-b33c-5b402b021576 | heat-stack-heat_server_port-o5rzphrabibu | fa:16:3e:7f:41:6f | ip_address='10.1.1.96',                  | ACTIVE |
       |                                      |                                          |                   | subnet_id='0037fdd9-d617-420a-ae0d-      |        |
       |                                      |                                          |                   | 3b83b3129f0a'                            |        |
       +--------------------------------------+------------------------------------------+-------------------+------------------------------------------+--------+                                                                                                       
       openstack router list
        
        +--------------------------------------+-------------+--------+-------+-------------+-------+----------------------------------+
        | ID                                   | Name        | Status | State | Distributed | HA    | Project                          |
        +--------------------------------------+-------------+--------+-------+-------------+-------+----------------------------------+
        | a9d1d8bb-a11f-4224-bcb6-f86636c41cba | test        | ACTIVE | UP    | False       | False | 6b667b69cd544138838b795557a00c60 |
        | e3d84406-754d-49d4-98ec-749241175288 | heat_router | ACTIVE | UP    | False       | False | 6b667b69cd544138838b795557a00c60 |
        +--------------------------------------+-------------+--------+-------+-------------+-------+----------------------------------+

        openstack volume list
        
        +--------------------------------------+-------------------------------------+-----------+------+--------------------------------------+
        | ID                                   | Name                                | Status    | Size | Attached to                          |
        +--------------------------------------+-------------------------------------+-----------+------+--------------------------------------+
        | affb3369-c3ea-4f1f-bef5-96f932142d2c | heat-stack-heat_volume-q6itn5uugwtp | in-use    |   20 | Attached to heat_server on /dev/sdb  |
        | cb7f8f2e-d5bd-42e4-bcb4-b62f63dcc8a6 | volume-stack-volume-01-26jz4xci46ia | available |   20 |                                      |
        +--------------------------------------+-------------------------------------+-----------+------+--------------------------------------+

In this way the resources gets created in openstack env using heat-stack.
