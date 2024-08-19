# lil_bind


## Usage

```yaml
  - name: get_dns_ip
      register: telepod_ip
      debug:
        msg: #                     -------- >> just an example how you can get secrets and use it with lil_bind << ---------
          "{{ lookup('community.hashi_vault.vault_kv2_get', 'workshop' , engine_mount_point='keyvalue', url='http://127.0.0.1:8200',  token=vault_token)['data']['data']['telepod_ip'] }}"
     - name: install podman
       ansible.builtin.yum:
         name:
         - podman
           state: latest

    - name: lil_bind #          --------- >> we use a block, so we just define the vars once and some are needed by multiple roles << ---------
      vars:        
          domains: [
            {domain: "pod.com", ip: "{{telepod_ip.msg}}",  # ----------- >> we define a few domains and subdomains  << -----------
              sub_domains: [{sub_domain: "tele", ip: "{{telepod_ip.msg.split('.')[-1]}}"}]},
            {domain: "test.com", ip: "192.168.2.120",
              sub_domains: [{sub_domain: "test", ip: "120"}]}
            ]        
          forwarders: [100.100.100.100]                 # ----------- >> needed for named.conf.local & named.conf.options  << -----------
          subnets: [192.168.0.0/16,100.0.0.0/8]
          allow_queries: ["localhost","192.168.0.0/16","100.0.0.0/8"]
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
