1. **Specify Providers**:
    ```yaml
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