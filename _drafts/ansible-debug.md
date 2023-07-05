Sometimes it is very usefull to use debugging techniques to troubleshoot a non-working ansible playblook.

When you run a task in Ansible, the entire output of the task will not be printed to stdout(Terminal). If you wish to see the output of the task, then you have to store it in the register variable and then later print it.

## Register the output
```yaml
tasks:
  - name: First Task - Using shell module to check if virtualenv is present or not
    ansible.builtin.shell:
      cmd: which virtualenv 
    register: virtualenv_output
    ignore_errors: True

  - name: Second Task - Print the full output
    ansible.builtin.debug:
      var: virtualenv_output

```
Remember that you can check if syntax is correct with **ansible-playbook --syntax-check**, after running **ansible playbook** you can take a look to whatever the first task had returned

The output contains some standard data and some additional provided by the module used. The standard return values are [here](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html)

### Accessing specific attributes
```yaml
   - name: Just checking the exit code 
     ansible.builtin.debug:
       msg: "{{ virtualenv_output.rc }}"
   - name: Just checking the exit code - Python dict way
     ansible.builtin.debug:
       msg: "{{ virtualenv_output['rc'] }}"
```

### Writing output to file
```yaml
   - name: Reroute the output to a file
     ansible.builtin.copy:
       content: "{{virtualenv_output}}"
       dest: "/tmp/virtualenv_output"
```

## Decision Making
Execute a block only in specific condition
```yaml
   - name: Install the package based on the return code
     ansible.builtin.apt:
       pkg: python3-virtualenv
       state: present
     when: virtualenv_output.rc != 0
     register: virtualenv_install_output
```

## Iteration
Accumulation of results of outputs per iteration, check the debug section: **It contains the item and the return code per loop**
```yaml
   - name: Using loops
     ansible.builtin.shell:
       cmd: rm -f "{{item}}"
     register: removed_output
     loop:
       - test_file.txt
       - abc.txt

   - name: Print the removed output
     ansible.builtin.debug:
       msg:
         - Return code for {{removed_output.results.0.item}} is {{removed_output.results.0.rc}}
         - Return code for {{removed_output.results.1.item}} is {{removed_output.results.1.rc}}
```
