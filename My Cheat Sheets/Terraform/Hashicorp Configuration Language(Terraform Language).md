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
    2. _Default_ field is optional. If not mentioned, terraform will prompt for values at runtime.
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
        1. **Int**:
            ```ini
            # Usage -----> var.example
            variable "example"{ 
            default     = 0
            type        = int
            }
            ```
        2. **String**:
            ```ini
            # Usage -----> var.example
            variable "example"{ 
            default     = "example"
            type        = string
            }
            ```
        3. **Lists**:
            ```ini
            # Usage -----> var.example[index] # Index starts at 0.
            variable "example"{ 
            default     = ["example", "example1", "example2"]
            type        = list(string) # Strict Typechecking
            }
            ```
        4. **Maps**:
            ```ini
            # Usage -----> var.example["key"]
            variable "example"{ 
            default     = {
                "color" = "Brown"
                "name"  =  "Charlie" }
            type        = map(string) # Strict Typechecking
            }
            ```
        5. **Sets**: Must contain unique elements.
            ```ini
            # Usage -----> var.example(index) # Index starts at 0.
            variable "example"{ 
            default     = [1, 2, 3 ]
            type        = set(int) # Strict Typechecking
            }
            ```
        6. **Objects**: used to create complex var using other var types.
            ```ini
            # Usage -----> var.example[key]
            variable "custom"{ 
            # Define the type of the object here.
            type = object({
                name    = string
                color   = string
                age     = int
                food    = list(string)
                fav_pet = bool
            })
            default = {
                name    = "bella"
                color   = "Blue"
                age     = 3
                food    = [ "Fried Chicken", "Biryani" ]
                fav_pet = true
            }
            }
            ```
        7. **Tuple**: Heterogenous list of various datatypes. Type and order the of element and No of elements should match. This what differentiates list from tuple.
            ```ini
            # Usage -----> var.example[key]
            variable "example"{ 
            default     = [1, "Hello", true ]
            type        = tuple([int, string, bool]) # Type and length should match
            }
            ```
            
5. **Output vars**: Terraform also supports output variables.
    - Output variables are printed after `terraform apply`.
    - All Output variables can be printed using `terraform output`.
    - Individual variables can be printed using `terraform output variable_name`
    ```ini
    output 'var-name'{
        value = "Mandatory argument. This usually is a reference expression"
        description = "Optional"
    }
    ---
    resource "random_pet" "my-pet" {
      length    = var.length 
    }
    
    output "pet-name" {
        value = random_pet.my-pet.id
        description = "Record the value of pet ID generated by the random_pet resource"
    }
    
    resource "local_file" "welcome" {
        filename = "/root/message.txt"
        content = "Welcome to Kodekloud."
    }
    output "welcome_message" {
      value = local_file.welcome.content
    }
    ```