## How Terraform Applies a Configuration

When Terraform creates a new infrastructure object represented by a `resource` block, the identifier for that real object is saved in Terraform's [state](https://developer.hashicorp.com/terraform/language/state), allowing it to be updated and destroyed in response to future changes. For resource blocks that already have an associated infrastructure object in the state, Terraform compares the actual configuration of the object with the arguments given in the configuration and, if necessary, updates the object to match the configuration.

In summary, applying a Terraform configuration will:

- _Create_ resources that exist in the configuration but are not associated with a real infrastructure object in the state.
- _Destroy_ resources that exist in the state but no longer exist in the configuration.
- _Update in-place_ resources whose arguments have changed.
- _Destroy and re-create_ resources whose arguments have changed but which cannot be updated in-place due to remote API limitations.

This general behavior applies for all resources, regardless of type. The details of what it means to create, update, or destroy a resource are different for each resource type, but this standard set of verbs is common across them all.

The meta-arguments within `resource` blocks, documented in the sections below, allow some details of this standard resource behavior to be customized on a per-resource basis.