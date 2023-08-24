### What is a “Here Document”? 
A “here document”, also known as a "here doc", is **a special type of input redirection that allows a user to specify multiple lines of input for a command**. This is particularly useful for specifying long blocks of text or script code as input to a command.

```bash
command = <<DELIMITER
line 1
line 2
line 3
...
DELIMITER
-------------------------------
policy = <<EOF
line 1
line 2
line 3
...
EOF
```

Here, `DELIMITER` can be any string. `EOF` is most commonly used.