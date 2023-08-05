1. **Lookup plugins**:  retrieve data from outside sources such as files, databases, key/value stores, APIs, and other services. 
    1. Like all templating, lookups execute and are evaluated on the Ansible control machine. Ansible makes the data returned by a lookup plugin available using the standard templating system. 
    2. Before Ansible 2.5, lookups were mostly used indirectly in `with_<lookup>` constructs for looping. 
    3. Starting with Ansible 2.5, lookups are used more explicitly as part of Jinja2 expressions fed into the `loop` keyword.
2. #### ansible.builtin.csvfile lookup – read data from a TSV or CSV file
    1. The csvfile lookup reads the contents of a file in CSV (comma-separated value) format. The lookup looks for the row where the first column matches keyname (which can be multiple words) and returns the value in the `col` column (default 1, which indexed from 0 means the second column in the file).
    ```yaml
    - name:  Match 'Li' on the first column, return the second column (0 based index)
      ansible.builtin.debug: msg="The atomic number of Lithium is {{ lookup('ansible.builtin.csvfile', 'Li file=elements.csv delimiter=,') }}"

    - name: msg="Match 'Li' on the first column, but return the 3rd column (columns start counting after the match)"
      ansible.builtin.debug: msg="The atomic mass of Lithium is {{ lookup('ansible.builtin.csvfile', 'Li file=elements.csv delimiter=, col=2') }}"
    
    - name: Define Values From CSV File, this reads file in one go, but you could also use col= to read each in it's own lookup.
      ansible.builtin.set_fact:
        loop_ip: "{{ csvline[0] }}"
        int_ip: "{{ csvline[1] }}"
        int_mask: "{{ csvline[2] }}"
        int_name: "{{ csvline[3] }}"
        local_as: "{{ csvline[4] }}"
        neighbor_as: "{{ csvline[5] }}"
        neigh_int_ip: "{{ csvline[6] }}"
      vars:
        csvline = "{{ lookup('ansible.builtin.csvfile', bgp_neighbor_ip, file='bgp_neighbors.csv', delimiter=',') }}"
      delegate_to: localhost
    ```