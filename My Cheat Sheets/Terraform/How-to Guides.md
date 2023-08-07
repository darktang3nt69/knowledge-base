1. **Specify Providers**:
    ```ini
    terraform {
      required_providers {
        ansible = { 
          #Community ansible Provider
          source = "nbering/ansible"
          version = "1.0.4"
        }
      }
    }
    ```
1. **Resource Attributes**: Output of one resource can be used as input to another resource through this. 
    - _Format_ : **`${resource_type.resource_name.attributes}`**
    ```ini
     resource "time_static" "time_update"{
     }

     resource "local_file" "time"{
         filename = "/root/time.txt"
         content = "Time stamp of this file is ${time_static.time_update.id}"
     }
    ```
3. **Resource Dependency**:
    1. _Implicit Dependency_: Terraform knows which resources are dependent on others based `resource attributes linking` or `reference expressions` as in above example. So it creates the dependent resource first and then the other.
    2. _Explicit Dependency_: Dependency on other resource is explicitly defined. For example,
        ```ini
        resource "local_file" "pet"{
            filename   = "/root/pet.txt"
            content    = "I love cats"
            # Takes a list of dependent resource in the format: 'resource_type.resource_name'
            depends_on = [
            random_pet.my-pet
            ]
        }

        resource "random_pet" "my-pet"{
            prefix    = var.prefix
            seperator = var.separator
            length    = var.length
        }
        ```