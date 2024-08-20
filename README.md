# lil_bind Ansible Galaxy Collection 
  

---
lil_bind installs  Bind9 podman container and configures a ***static DNS*** for you

---
- one domain and ip per zone
- subdomains can be used, or just left blank
  
i dont need a dhcp for my iac stuff, but still need a dns, so it was not intendet to make use of dynamic updates since all my containers and vms have a static ip because of the nic's (bridges/nats) anyway, so i decided just to use a tailscale router and instead of dhcp ill use a little netkwork manager that gives ip by the given zone. So i just have a preset file that contains all my zones and subnets and the networkmanager will cycle ips in the given subnet and zone.


## Usage
- import collection and use root
      
    ```yaml
    
    - hosts: <your_host>
      gather_facts: no
      become: true
      become_method: sudo
      become_user: root
      collections:
        - ji_podhead.lil_bind 
    ```
    
- put all variables in a block because some are needed  for multiple roles and we dont like redundancy
  
    ```yaml
  tasks:
    - name: install podman
      ansible.builtin.yum:
        name:
        - podman
        state: latest
    - name: lil_bind
      vars:
          container_name: "dns"
          container_ip: "{{192.168.50.0}}"        
          dns_admin: "admin"
          dns_domain: "dns.com"
          domains: [
                      {
                        domain: "pod.com", ip: "192.168.3.0",
                        sub_domains: [{sub_domain: "tele", ip: 2}]
                      },
                      {
                        domain: "test.com", ip: "192.168.2.120",
                        sub_domains: [{sub_domain: "test", ip: "121"}]
                      }
                    ]        
          forwarders: [100.100.100.100]
          subnets: [192.168.0.0/16,100.0.0.0/8]
          allow_queries: ["localhost","192.168.0.0/16","100.0.0.0/8"]
      
    ```
    
- fire the collection
    ```yaml      
      block:
      - name: install bind9
        import_role: 
          name: ji_podhead.lil_bind.install

      - name: config
        import_role: 
          name: ji_podhead.lil_bind.config

      - name: set_zones
        import_role: 
          name: ji_podhead.lil_bind.set_zones
      
      - name: update & restart bind9
        import_role:
          name: ji_podhead.lil_bind.update
    ```
    
    --- 

## output
