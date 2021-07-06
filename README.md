# 08.02_work_in_playbook
---


    name: Install Java
    hosts: all
    tasks:
        name: Set facts for Java 11 vars
        set_fact:
        java_home: “/opt/jdk/{{ java_jdk_version }}”
        tags: java
        name: Upload .tar.gz file containing binaries from local storage
        copy:
        src: “{{ java_oracle_jdk_package }}”
        dest: “/tmp/jdk-{{ java_jdk_version }}.tar.gz”
        register: download_java_binaries
        until: download_java_binaries is succeeded
        tags: java
        name: Ensure installation dir exists
        become: true
        file:
        state: directory
        path: “{{ java_home }}”
        tags: java
        name: Extract java in the installation directory
        become: true
        unarchive:
        copy: false
        src: “/tmp/jdk-{{ java_jdk_version }}.tar.gz”
        dest: “{{ java_home }}”
        extra_opts: [–strip-components=1]
        creates: “{{ java_home }}/bin/java”
        tags:
            java
        name: Export environment variables
        become: true
        template:
        src: jdk.sh.j2
        dest: /etc/profile.d/jdk.sh
        tags: java
    name: Install Elasticsearch
    hosts: elasticsearch
    tasks:
        name: Upload tar.gz Elasticsearch from remote URL
        get_url:
        url: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz
        dest: /tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz
        mode: 0755
        timeout: 60
        force: true
        validate_certs: false
        register: get_elastic
        until: get_elastic is succeeded
        tags: elastic
        name: Create directrory for Elasticsearch
        file:
        state: directory
        path: “{{ elastic_home }}”
        tags: elastic
        name: Extract Elasticsearch in the installation directory
        become: true
        unarchive:
        copy: false
        src: “/tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz”
        dest: “{{ elastic_home }}”
        extra_opts: [–strip-components=1]
        creates: “{{ elastic_home }}/bin/elasticsearch”
        tags:
            elastic
        name: Set environment Elastic
        become: true
        template:
        src: templates/elk.sh.j2
        dest: /etc/profile.d/elk.sh
        tags: elastic
        name: Download Kebana
        get_url:
        url: https://artifacts.elastic.co/downloads/kibana/kibana-5.3.0-amd64.deb
        dest: /tmp/kibana-5.3.0-amd64.deb
        mode: 0755
        timeout: 60
        force: true
        validate_certs: false
        name: Install Kibana
        become: true
        shell: cd /tmp/ && dpkg -i kibana-5.3.0-amd64.deb
        tags: kibana,skip_ansible_lint
        name: Easiest config kibana
        template:
        src: templates/kibana.yml.j2
        dest: /etc/kibana/kibana.yml
        mode: 0555


