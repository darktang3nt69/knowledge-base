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


12. **String Manipulation:**
    1. **String Length:** Pattern: `${#var_name}`
        ```bash
         distro='kali'
         echo ${#distro}
         > 4   
        ```
    2. **String Concatenation:** Pattern: `str3=$string1$string2`
        ```bash
        str1="Mr. "
        str2="Robot"
        str3=$str1$str2
        echo $str3
        > Mr. Robot
        ```
    3. **Substring:** Pattern: `expr index "$supertring" "$searchstring"`
        ```bash
        str="Bash is cool."
        expr index "$str" "cool" # expr index "$supertring" "$searchstring"
        > 9
        ```
    4. **Extracting Substrings:** Pattern: `${var:start_index:stop_index}`
        ```bash
        foss="Fedora is a free operating system."
        echo ${foss:0:6} # get string from 0th(including) until 6th position(excluding).
        > Fedora
        echo ${foss:12} # get remaining string from 12th(including) position. 
        > free operating system.
        ```
     5. **Replacing Substring:** Pattern: `${string/replace/new}`
         ```bash
         foss="Fedora is a free operating system."
         echo ${foss/Fedora/Kali} # Does not modify the original string.
         > Kali is a free operating system.
         foss=${foss/Fedora/Kali} # Persist the changes. Store the changes in a var.
         ```
     6. **Deleting Strings:** Pattern: `${string/substr}`
         ```bash
         fact="Sun is a big star."
         echo ${fact/big}  -------->  # Remove 'big'
         > Sun is a  star.
         echo ${fact/ big} -------->  # Remove 'big '
         > Sun is a star.
         num=112-234535-456456-343
         echo ${num/-}    -------->   # Remove 1st occurence of '-'
         > 112234535-456456-343
         echo ${num//-}  -------->    # Remove all occurences of '-'
         > 112234535456456343
         num=${num//-} -------->      # Persist the changes
         echo $num
         > 112234535456456343
         ```
    7. **Uppercase/Lowercase**:
        ```bash
        legend='taylor swift'
        actor='EMMA STONE'
        echo ${legend^^} -----> # Make all characters caps.
        > TAYLOR SWIFT
        echo ${actor,,} ------> # Make all characters small.
        > emma stone
        echo ${legend^} ------> # Make 1st char caps.
        > Taylor swift
        echo ${actor,} ------->  # Make 1st char small.
        > eMMA STONE
        echo ${legend^^[ts]} ----> # Make all occurences of `t` and `s` caps.
        > Taylor SwifT
        echo ${actor,,[ES]} -----> # Make all occurences of `E` and `S` small.
        eMMA sTONe 
        ```

13. **Decision Statements**:
    1. **if:**
        ```bash
        if [ condition ]; then
            your code
        fi
        # ex:
        if [ $(whoami) = 'root' ]; then
            echo 'You are root.'
        fi
        ```
    2. **if-else:**
        ```bash
        if [ condition ]; then
            your code
        else 
            echo 'You are not root'
        fi
        # ex:
        if [ $(whoami) = 'root' ]; then
            echo 'You are root.'
        else
            echo 'You are not root'
        fi
        ```
    3. **elif:**
        ```bash
        #!/bin/bash
        age=69
        if [ $age -lt 13 ]; then
                echo 'You are a kid.'
        elif [ $age -lt 20 ]; then
                echo 'You are a teen.'
        elif [ $age  -lt 65 ]; then
                echo 'You are an adult.'
        else
                echo 'You are a senior citizen.'
        fi
        ```
    4. **Nested if:**
        ```bash
        #!/bin/bash
        temp=$1
        
        if [ $temp -gt 5 ]; then
                if [ $temp -lt 15 ]; then
                        echo 'Weather is cold'
                elif [ $temp -lt 25 ]; then
                        echo 'Weather is Nice'
                else
                        echo 'Weather is hot'
                fi
        else
                echo 'It is COLD!!!!'
        fi
        ```
    5. **case:**
        ```bash
        #!/bin/bash
        char=$1
        case $char in
                [a-z])
                        echo 'Small Alphabets.' ;;
                [A-Z])
                        echo 'Caps.' ;;
                [0-9])
                        echo 'Numbers.' ;;
                *)
                        echo 'Special Chars.' ;;
        esac
        ```
    6. 