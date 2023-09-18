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
    # Command chaining:
    distro=$(cat /etc/os-release | egrep '^NAME=')
    ```
8. Command Line Arguments:
    ```bash
    script.sh:
    -----------------------
    #!/bin/bash
    wcount1=$(wc -l < $1)
    wcount2=$(wc -l < $2)
    wcount3=$(wc -l < $3)
    echo "There are $wcount1 lines in $1."
    echo "There are $wcount2 lines in $2."
    echo "There are $wcount3 lines in $3."
    -----------------------
    In terminal:
    ./script.sh /etc/os-release /etc/passwd /etc/hosts # pass the arugments while calling the script, separated by space.
    ```
9. **Special Bash Variables:**

| Special Variables | Description                                         |
| ----------------- | --------------------------------------------------- |
| `$0`              | Name of the bash script. Based on how it is called. |
| `$!`,`$2`...`$n`  | Bash script Arguments.                              |
| `$$`              | Process ID of the current shell.                    |
| `$#`              | Total no. of arguments passed to the script.        |
| `$@`              | Value of all arguments passed to the script.        |
| `$?`              | Exit status of the last executed command.           |
| `$!`              | Process ID of the last executed command.            |
|                   |                                                     |


10. **Arrays:**
    1. Declaring Arrays. Index starts from `0` ----> `n-1`:
        ```bash
        files=("f1.txt", "f2.txt", "f3.txt", "f4.txt", "f5.txt", "f6.txt")
        ```
    2. Accessing Arrays:
        ```bash
        # 1. Accessing using index value:
        echo "${files[4]}, ${files[3]}"
        
        # 2. Unpack the array:
        echo ${files[*]}

        # 3. Array length:
        echo ${#files[@]}

        # 4. Re-assign values of array:
        files[4]="69.txt"
        ```
    3. Append Array Elements:
        ```bash
        files+=("696969.txt")
        ```
    4. Deleting Array elements/Array:
        ```bash
        num=(1,2,3,4,5)

        # Delete at index variable:
        unset num[2]
        
        # Delete entire Array:
        unset num # Without index value, it deletes the array.
        ```
    5. Heterogenous Array:
        ```bash
        hetero=('fsociety', 69, 'etc', 3.1412)
        ```


11. **Arithmetic Operations:**
    1. **Addition and Subtraction:**
        ```bash
        echo $(( 1 + 2 ))
        echo $(( 70 - 1 ))
        ```
    2. **Multiplication and Division:**
        ```bash
        # 1. Multiplication
        echo $(( 1024 * 1024 ))
        
        # 2. Division
        echo $(( 5/2 )) ----> output will be `2` instead `2.5`.
        echo "5/2"  | bc -l -----> output: 2.50000000000000000000 # More accurate.
        ```
    3. **Powers and Remainders:**
        ```bash
        # 1. Powers
        echo $(( 2**3 )) -----> output: 8
        
        # 2. Remainders
        echo $(( 17%5 )) ------> output: 2
        ```
    4. Convert from Celsius to Fahrenheit:
        ```bash
        echo "scale=2; $celsius * (9/5) + 32" | bc -l 
        ```