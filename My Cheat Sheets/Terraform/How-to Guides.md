###  **Specify Providers**:
```hcl
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

### **Resource Attributes**: 
- Output of one resource can be used as input to another resource through this. 
 - _Format_ : **`${resource_type.resource_name.attributes}`**
 ```hcl
 resource "time_static" "time_update"{
     }

     resource "local_file" "time"{
         filename = "/root/time.txt"
         content = "Time stamp of this file is ${time_static.time_update.id}"
     }
```
 
### **Resource Dependency**:

1. _Implicit Dependency_: Terraform knows which resources are dependent on others based `resource attributes linking` or `reference expressions` as in above example. So it creates the dependent resource first and then the other.
2. _Explicit Dependency_: Dependency on other resource is explicitly defined. For example,
```hcl
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

### **for_each**:
```hcl
# Variable.tf
variable "users" {
    type = list(string)
    default = [ "/root/user10", "/root/user11", "/root/user12", "/root/user10"]
}
variable "content" {
    default = "password: S3cr3tP@ssw0rd"
}
```

```hcl
resource "local_sensitive_file" "name" {
    filename = each.value
    for_each = toset(var.users)
    content = var.content
}
```

### **count**:
`count` is a meta-argument defined by the Terraform language. It can be used with modules and with every resource type.

The `count` meta-argument accepts a whole number, and creates that many instances of the resource or module. Each instance has a distinct infrastructure object associated with it, and each is separately created, updated, or destroyed when the configuration is applied.
- [`count.index`](https://developer.hashicorp.com/terraform/language/meta-arguments/count#count-index) — The distinct index number (starting with `0`) corresponding to this instance.

```hcl
resource "aws_instance" "server" {
  count = 4 # create four similar EC2 instances

  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${count.index}"
  }
}
```

Ex:
```hcl
iam-user.tf
---
resource "aws_iam_user" "users" {
  name  = var.project-sapphire-users[count.index]
  count = length(var.project-sapphire-users)
}
```

```hcl
variable.tf
---
variable "project-sapphire-users" {
  type    = list(string)
  default = ["mary", "jack", "jill", "mack", "buzz", "mater"]
}
```

### **Include a file in tf file.**:
- Using, `file` function, we can include or reference external files in tf file. else we need to use heredoc syntax.
    ```hcl
    resource "example_file" "example"{
        file_content = file(filename)
    }
    ```