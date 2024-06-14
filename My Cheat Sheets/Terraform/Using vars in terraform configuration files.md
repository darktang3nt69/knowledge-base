1. **_Interactive Mode_**: If _default_ is not specified in var block, Terraform prompts for values at runtime.
2. _**variables.tf file**_: variables can be declared here as variable blocks. Automatically loaded.
3. **_Cmd Line_**: We can also pass vars through cmd line flags:
    ```bash
    terraform apply -var "filename=/root/pet.txt" -var "color=blue"
    ```
3. **_Env Variables_**: set as env vars
    ```bash
    export TF_VAR_variable_name="example"
    ```
4. **_Variable Definition Files_**:  Variables can be declared in external filles as well.
    1. Any file(s) named as below are automatically loaded:
        - `terraform.tfvars`, or
        - `terraform.tfvars.json` or
        - `*.auto.tfvars`, or
        - `*.auto.tfvars.json`
    2. Any other file name ending with `*.tfvars` or `*.tfvars.json`  needs to be included during `terraform apply` as:
        ```bash
        terraform apply --var-file variables.tfvars
        ```
        
1. **_Variable Definition Precedence_**: Terraform loads variables in the following order, with later sources taking precedence over earlier ones:

    Order | Option
    ---|---
    1 | Env variables (Lowest)
    2 | terraform.tfvars
    3 | *.auto.tfvars (Alphabetical Order)
    4 | -var or --var-file (Command-line args) (Highest)