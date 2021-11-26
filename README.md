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
  4. Lets create single volume in openstack env by running `heat-volume-stack.yaml` file
     `openstack stack create -t heat-volume-stack.yaml <stack_name>`
     ```
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
     ```    
     This will create a single volume of 10Gb size as follows: 
     `openstack volume list`
        ```
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | ID                                   | Name                                | Status    | Size | Attached to |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | cb7f8f2e-d5bd-42e4-bcb4-b62f63dcc8a6 | volume-stack-volume-01-26jz4xci46ia | available |   10 |             |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        ```
     To resize volume change the `default` size in heat-volume-stack.yaml file to required value:
     ```
     parameters:
       heat_volume_size:
         type: number
         label: Volume Size (GB)
         description: External Volume size in GB
         default: 20
     ```
     Run `openstack stack update -t heat-volume-stack.yaml <stack-name>`
     ```
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
     ```
     ```
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | ID                                   | Name                                | Status    | Size | Attached to |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
        | cb7f8f2e-d5bd-42e4-bcb4-b62f63dcc8a6 | volume-stack-volume-01-26jz4xci46ia | available |   20 |             |
        +--------------------------------------+-------------------------------------+-----------+------+-------------+
     ```
  5. Create server with resources such as network, volume, floating ip, ports etc.
     `openstack stack create -t heat-resources-stack.yaml <stack_name>`
     
     Similarly, as we created volume using heat template this script will create all the necessary resources along with server.
     
    
