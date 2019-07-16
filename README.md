# Ansible-Splunk-Forwarder-Deployment

---
- hosts: all
  tasks:
    - name: Copy Splunk Univrsal Forwarder to hosts
      remote_user: root
      become: yes
      become_method: sudo
      copy:
         src: /etc/ansible/Testing/splunkforwarder-7.3.0-657388c7a488-linux-2.6-x86_64.rpm
         dest: /root
         owner: root
         group: root
         mode: 0644

    - name: Install Splunk Forwarder to hosts
      remote_user: root
      yum:
       name: /root/splunkforwarder-7.3.0-657388c7a488-linux-2.6-x86_64.rpm
       state: present

    - name: Copy the Splunk Forwarder inputs config to hosts
      remote_user: root
      copy:
        src: /etc/ansible/Testing/inputs.conf
        dest: /opt/splunkforwarder/etc/system/local/inputs.conf
        directory_mode: yes
        owner: splunk
        group: splunk
        mode: 0600

    - name: Start Splunk forwarder service.
      remote_user: root
      become_user: splunk
      shell: /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes --no-prompt --seed-passwd pass

    - name: Enable Splunk forwarder on boot
      remote_user: root
      become_user: splunk
      shell: /opt/splunkforwarder/bin/splunk enable boot-start

    - name: Add usher03 as indexer
      remote_user: root
      become_user: splunk
      shell: /opt/splunkforwarder/bin/splunk add forward-server X.X.X.X:9997 -auth admin:pass
      ignore_errors: yes

    - name: Restart Splunk universal forwarder
      remote_user: root
      shell: /opt/splunkforwarder/bin/splunk restart

    - name: Check Splunk forwarder service.
      remote_user: root
      command: /opt/splunkforwarder/bin/splunk status
      register: service_splunk_status
     
    - name: Report Splunk forwarder Status.
      remote_user: root
      debug:
         var: service_splunk_status.stdout_lines
...
