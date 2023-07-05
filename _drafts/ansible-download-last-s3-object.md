➜  users ansible-galaxy collection install amazon.aws

    - name: List keys simple
      amazon.aws.s3_object:
        bucket: s3://e2-production/macro/
        mode: list
      register: output_list
    - name: Print return information from the previous task
      ansible.builtin.debug:
        var: output_list
