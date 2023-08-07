1. The main purpose of the Terraform language is declaring resources, which represent infrastructure objects.
2. A _Terraform configuration_ is a complete document in the Terraform language that tells Terraform how to manage a given collection of infrastructure. A configuration can consist of multiple files and directories.
3. The syntax of the Terraform language consists of  only a few basic elements:
    1. **Blocks**: 
        1. _Blocks_ are containers for other content and usually represent the configuration of some kind of object, like a resource. 
        2. Blocks have a _block type,_ can have zero or more _labels,_ and have a _body_ that contains any number of arguments and nested blocks. 
        3. Most of Terraform's features are controlled by top-level blocks in a configuration file.
    2.  _Arguments_: assign a value to a name. They appear within blocks.
    3.  _Expressions_ represent a value, either literally or by referencing and combining other values. They appear as values for arguments, or within other expressions.
        ```ini
        resource "local_file" "pet" {
          filename = ~/pet.txt
          content  = 'I love cats!!'
        }
        
        <BLOCK NAME> "<BLOCK RESOURCE>" "<Name>" {
          #Block body
          <IDENTIFIER> = <EXPRESSION> # Argument
        }
        ```
        
4. **Resource Block**: This block specifies the _resources_ that terraform manages.
    ```ini
    # Template for Resource Block.
    <Resource> "<PROVIDER_RESOURCE>" "<RESOURCE_NAME>" {
          #body
          <IDENTIFIER> = <EXPRESSION> # Argument
          ...
          ...
        }
        
    # blockname = resource
    # provider  = local
    # resource  = file
    # label     = pet
    # arguments = filename and content
    resource "local_file" "pet" {
          filename = ~/pet.txt
          content  = "I love cats!!"
        }
    ```
    
1. **Variable block**: This declares the variables.
    1. Can be declared in `variables.tf` file and can used in our script.
    ```ini
    #Template:
    variable "var_name" {
    default     = "something"
    type        = string || int || bool || map || list || set || object || tuple || any(default) # Optional
    description = "something something" # Optional
    }

    # Example:
    variable "filename"{
    default     = '/root/pets.txt'
    type        =  string
    description = "path of the local file"
    }
    
    # Usage in scripts:
    resource "local_file" "example"{
    filename = var.filename # var.variable_name
    ...
    }
    
    ```
    2. **Types of vars**:
        1. **int**:
            ```ini
            # int
            # Usage -----> var.example
            variable "example"{ 
            default     = 0
            type        = int
            }
            ```
        2. **string**:
            ```ini
            # int
            # Usage -----> var.example
            variable "example"{ 
            default     = 0
            type        = int
            }
            ```