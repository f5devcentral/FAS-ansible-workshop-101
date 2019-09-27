# Use Case

Deploy a virtual server that will re-direct HTTP requests to HTTPS using an iRule

# Pre-requisites

<< Provide link to proivioner and lab inventory file to let user know how to get the BIG-IP and VIP info etc >> and specify that is what we are using for the environment

# Solution

We are deploying the following
- Two backend application servers running apache
- One pool which consists on the two appplication servers
- One virtual server listening on port 443
- Another virtual server listening on port 80. A redirect iRule is attached to the virtual server which redirect HTTP traffic to HTTPS

```
- name: Creating HTTPS application
  connection: local
  gather_facts: false
  hosts: lb

  vars:
   pool_name: 'http-pool'
   vip_name: 'myApplication"
   
  tasks:
      
  - set_fact:
     provider:
      server: "{{private_ip}}"
      user: "{{ansible_user}}"
      password: "{{ansible_ssh_pass}}"
      server_port: 8443
      validate_certs: no

  - name: Create node
    bigip_node:
      provider: "{{provider}}"
      host: "{{hostvars[item].ansible_host}}"
      name: "{{hostvars[item].inventory_hostname}}"
    loop: "{{ groups['webservers'] }}"
    
  - name: Create pool
    bigip_pool:
      provider: "{{provider}}"
      name: "{{pool_name}}"
      lb_method: "round-robin"
      monitors: "/Common/http"
      monitor_type: "and_list"
    
  - name: Add members to pool
    bigip_pool_member:
      provider: "{{provider}}"
      state: "present"
      name: "{{hostvars[item].inventory_hostname}}"
      host: "{{hostvars[item].ansible_host}}"
      port: "80"
      pool: "{{pool_nme}}"
    
  - name: Create Virtual Server
    bigip_virtual_server:
       description: "Secure web application"
       provider: "{{provider}}"   
       name: "{{vip_name}}"
       destination: "{{private_ip}}"
       port: 443
       snat: "Automap"
       all_profiles:
       - http
       - clientssl
       pool: "{{pool_name}}"
    
  - name: Create a redirect virtual server
    bigip_virtual_server:
       description: "Redirect Virtual server"
       provider: "{{provider}}"   
       name: "http_redirect"
       destination: "{{private_ip}}"
       port: 80
       all_profiles:
       - http
       all_rules:
       - _sys_https_redirect
```

# Testing the solution
<<  GIVE screen shots  >>

- Login to BIG-IP "<< >>" check the configuration deployed
- Access the application on VIP "<< >>:443"
- Access the application on VIP "<< >>:80" this will be recirected to port 443 

# Alternative solution if any
