# Ansible Collection - rytis.hashicorp

## Nomad - rytis.hashicorp.nomad

Example playbook:

    ---
    - hosts: all
      become: yes
    
      vars:
        master_groups:
          - master
          - both
    
      tasks:
        - name: Configure firewall
          ansible.posix.firewalld:
            port: "{{ item }}"
            permanent: yes
            state: enabled
          loop:
            - "4646/tcp"
            - "4647/tcp"
            - "4648/tcp"
            - "4648/udp"
          notify:
            - restart firewalld
    
        - name: discover servers
          ansible.builtin.set_fact:
            _masters: "{{ _masters | default([]) + groups[item] | default([]) }}"
          loop: "{{ master_groups }}"
    
        - name: get ip details
          ansible.builtin.set_fact:
            _masters_ips: "{{ _masters_ips| default([]) + [hostvars[item].ansible_default_ipv4.address] }}"
          loop: "{{ _masters }}"
    
        - name: Setup Nomad
          ansible.builtin.include_role:
            name: rytis.hashicorp.nomad
          vars:
            hashicorp_nomad_master_ips: "{{ _masters_ips }}"
    
      handlers:
        - name: restart firewalld
          ansible.builtin.service:
            name: firewalld
            state: restarted
