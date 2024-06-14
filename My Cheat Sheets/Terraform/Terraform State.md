Terraform state is a fundamental concept in Terraform's design that helps manage the relationship between your declared configuration and the actual resources provisioned in your infrastructure. The Terraform state file keeps track of the current state of your infrastructure as defined by your Terraform configuration files. Here's everything you need to know about Terraform state:

1. **What is Terraform State?**
   - The Terraform state is a JSON-format file that stores information about all the resources created by Terraform in a particular workspace (project) and their current state.
   - This state includes details such as resource IDs, attributes, metadata, dependencies, and any other information needed to manage and maintain your infrastructure.

2. **Purpose of Terraform State:**
   - Terraform uses the state to determine the differences between the desired infrastructure state defined in your configuration files and the current actual state of the infrastructure.
   - When you run `terraform plan`, Terraform compares the current state with the desired state to generate a plan of actions needed to bring the infrastructure into the desired state.
   - When you run `terraform apply`, Terraform makes API calls to the cloud providers to create, update, or delete resources based on the plan generated.

3. **Managing Terraform State:**
   - Terraform supports multiple methods for storing and managing state:
     - **Local State:** The default behavior is to store the state file locally in the working directory. However, this isn't recommended for production use, as it can lead to state file inconsistencies when working collaboratively.
     - **Remote Backends:** To overcome the limitations of local state, Terraform provides support for remote backends like Amazon S3, Azure Blob Storage, Google Cloud Storage, and more. Remote backends provide centralized storage for the state and support features like state locking for collaboration.
   
4. **State Locking:**
   - State locking is crucial when working with a team to prevent concurrent modifications to the same state file, which could lead to inconsistencies.
   - Remote backends provide state locking mechanisms to prevent multiple users from modifying the same infrastructure state simultaneously.

5. **State File Security:**
   - The state file often contains sensitive information, such as resource IDs and attributes, API tokens, and more.
   - Ensure that your state file is kept secure and not exposed to unauthorized users. This is particularly important when using remote backends, as the state file is stored on a shared storage system.

6. **State Management Commands:**
   - `terraform state list`: Lists all the resources and their addresses present in the state.
    ```bash
    # returns all the resources in the state file
    terraform state list [options] [address]

    # returns the matching resources if present in the state file
    terraform state list [resource_name]
    ```
   - `terraform state show <resource_address>`: Shows detailed information about a specific resource in the state.
    ```bash
    # returns all the info about the matching resource if present
    terraform state show [options] [address]
    ```
   - `terraform state rm <resource_address>`: Removes a resource from the state (use with caution).
   - `terraform state mv <old_resource_address> <new_resource_address>`: Moves a resource to a new address in the state.
    ```bash
    # 1. renames the resource if the source and destination are in the same state file.
    # 2. Can be used to move resources from one state file to another.
    # 3. Have to manually rename/move the resources to the moved/renamed versions.
    terraform state mv [options]  SOURCE DESTINATION
    ```

Please Refer to [[Terraform CLI#State Manipulation|State Management in terraform cli]].

Understanding and managing Terraform state is crucial for successful infrastructure management. Using remote backends and implementing proper state locking mechanisms will help you work collaboratively and avoid potential conflicts when managing infrastructure changes.