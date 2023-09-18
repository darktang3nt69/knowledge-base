1. **Shebang** has to present at the start of the script, if not, default shell of the user will interprets the script:
    ```bash
    #!/bin/bash
    ```
2. **Read Input**:
    ```bash
    #!/bin/bash
    read var_name
    ```
3. Use **Vars**:
    ```bash
    #!/bin/bash
    read var_name
    echo "Your name is $var_name"
    ```
3. **Single quotes** ('') is basic string, **Double quotes**("") is f-string.
4. Bash is **untyped**. No datatype in bash.
5. **Constant vars**:
    ```bash
    readonly PI=3.141
    ```
7. **Command Substitution**:
    ```bash
    # 1. Using $ Note: Just use this, It supports nested commands:
    date=$(date)
    # 2. Using ``:
    date=`date`
    # Command chaining
    distro=$(cat /etc/os-release | egrep '^NAME=')
    ```
8. Command Line Arguments:
    ```bash
    script.sh:
    -----------------------
    #!/bin/bash
    wcount=$(wc -l < $1)
    echo "There are $wcount lines in $1."
    -----------------------
    In terminal:
    ./script.sh /etc/os-release # pass the arugments while calling the script, separated by space.
    ```