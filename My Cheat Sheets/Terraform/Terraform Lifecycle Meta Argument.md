### **[Lifecycle Rules](<scalr.com/blog/lifecycle-meta-argument#:~:text=Every%20resource%20that%20is%20managed,)%2C%20Update%2C%20and%20Destroy.>)**: 
1. Every resource that is managed by Terraform has a lifecycle, this lifecycle contains three stages; `Apply (Create)`, `Update`, and `Destroy`. 
2. The `Apply` stage is where the resource is actually created by Terraform, 
3. `Update` is when a change is made to a preexisting resource - this might be incremental or a full recreate.
4. Finally, `Destroy` where the resource is removed from the environment. 
5. Sometimes however we may want to control these stages of the resources lifecycle a little more for this Terraform gives us the `lifecycle` meta-argument.

### **What can we control?**
Terraform gives us the following options that we can use in the `lifecycle` meta-argument:

- `create_before_destroy` — when an in-place update has to occur Terraform will create the new instance prior to destroying the old.
- `prevent_destroy` — do not allow the destroy flow to actually destruct the resource.
- `ignore_changes` — ignore any changes on specified fields or an entire object.
- `replace_triggered_by`
- `precondition` — check some thing before performing the action on the resource.
- `postcondition` — validate some thing after performing an action on the resource.
- for the `*condition` you can check out this [post](https://www.scalr.com/blog/terraform-v1-features).
### Create Before Destroy (bool)

- By default, when Terraform must change a resource argument that cannot be updated in-place due to remote API limitations, Terraform will instead destroy the existing object and then create a new replacement object with the new configured arguments.
- The `create_before_destroy` meta-argument changes this behavior so that the new replacement object is created _first,_ and the prior object is destroyed after the replacement is created.
- Note that Terraform propagates and applies `create_before_destroy` meta-attribute behaviour to all resource dependencies.
- For example, if resource A with enabled `create_before_destroy` depends on resource B with disabled `create_before_destroy` (by default), then Terraform enables `create_before_destroy` for resource B implicitly and stores it to the state file. You cannot override `create_before_destroy` to `false` on resource B, because that would imply dependency cycles in the graph.

Using the [Scratch provider](https://registry.terraform.io/providers/BrendanThompson/scratch/0.3.0) we can mock out an example of this:

```hcl
resource "scratch_string" "this" {
  in = "create_before_destroy"

  lifecycle {
    create_before_destroy = true
  }
}
```

The above will now ensure that in the event this resource is required to be replaced in- place that it will create the new instance first.

### Prevent Destroy (bool)

- This meta-argument, when set to `true`, will cause Terraform to reject with an error any plan that would _destroy_ the infrastructure object associated with the resource, as long as the argument remains present in the configuration.. 
- On `destroy` the resource would be removed from state but still _exist_ in the real world. 
- This can be used as a measure of safety against the accidental replacement of objects that may be costly to reproduce, such as database instances. 
- However, it will make certain configuration changes impossible to apply, and will prevent the use of the `terraform destroy` command once such objects are created, and so this option should be used sparingly.
- **Gotcha Point**: Since this argument must be present in _configuration_ for the protection to apply, note that this setting does not prevent the remote object from being _destroyed_ if the `resource` block were removed from _configuration_ entirely: in that case, the `prevent_destroy` setting is removed along with it, and so Terraform will allow the _destroy_ operation to succeed.
Let’s dive into an example to better understand this concept.

```hcl
resource "azurerm_resource_group" "this" {
  name     = "rg-prod"
  location = "australiasoutheast"

  lifecycle {
    prevent_destroy = true
  }
}
```

- In the above we are creating a resource group, and we have informed Terraform we want to prevent its destruction through the `lifecycle` meta-argument. 
- In this scenario let’s assume that we are only managing a portion of the resources within the resource group (RG) via Terraform and other via some other mechanism. 
- If we were to not have `prevent_destroy` when we eventually did a destruction those resources created out of Terraform would also be destroyed. 
- By having `prevent_destroy` we are now required to be more assertive when we want to destroy the RG, we would either have to remove it manually or commit a change removing the `lifecycle` attribute.
- I find that `prevent_destroy` is a favorite to security folks as it helps to add an extra level of assurance around destructive operations, especially on resource types that have such a large blast area like a resource group.
- `terraform destroy` would indeed destroy the resources.

### Ignore Changes (list of attribute names)

- Now we come to one of the more commonly used and in my opinion the most dangerous, `ignore_changes`. 
- A reason why you might want to use `ignore_changes` is if some outside force / process is going to be modifying your resources. 
- In order to make Terraform share management responsibilities of a single object with a separate process, the `ignore_changes` meta-argument specifies resource attributes that Terraform should ignore when planning updates to the associated remote object.
- Only attributes defined by the resource type can be ignored. `ignore_changes` cannot be applied to itself or to any other meta-arguments.
- An example, of this might be mutation of tags or tag values via Azure policy or potentially the number of instances of a resource due to a scaling event. 
- Both of those examples are what I would consider good reasons to utilize `ignore_changes`. Lets look at a very basic example:

```hcl
resource "aws_instance" "example" {
  # ...

  lifecycle {
    ignore_changes = [
      # Ignore changes to tags, e.g. because a management agent
      # updates these based on some ruleset managed elsewhere.
      tags,
    ]
  }
}

```

- In the above `scratch_block` we are ignoring **any** changes to the `in` block, and we cannot ignore a specific property on that block as the block is actually represented as a `set` which does not have an index or referenceable value. 
- It is important to note that the values provided here **must** be static, you cannot pass in a variable or a splat. 
- The only exception is the use of the special `all` keyword in place of a list which will then ignore all attributes on the resource.
- `ignore_changes` becomes dangerous when you start ignoring entire resources as then the changes you make to the code won’t alter the resource this means you’re only managing two stages of the resource `Apply` and `Destroy` all alteration would then have to be managed by an external system.

Many organisations uses tags for managing or attributing cost with cloud resources so ignore changing to particular tags or the tags property can be very valuable as it allows an external system to manage the tags on resources for you without Terraform overwriting the changes. When using `ignore_changes` my advice to be as specific about the property you’re wanting to ignore as you possibly can be!

### Replace Triggered By (list of resource or attribute references) - _Added in Terraform 1.2._

`replace_triggered_by` is a very new addition to language, only coming out with Terraform v1.2, it is also a very powerful argument. This will replace a particular resource based on another resource. Below is an example:

```hcl
resource "aws_appautoscaling_target" "ecs_target" {
  # ...
  lifecycle {
    replace_triggered_by = [
      # Replace `aws_appautoscaling_target` each time this instance of
      # the `aws_ecs_service` is replaced.
      aws_ecs_service.svc.id
    ]
  }
}

```

In the above example we have two resources `scratch_bool.this` and `scratch_string.this`and we are trying to  replace  `scratch_bool.this`. What this means is that if we were to update `scratch_bool.this.in` to be `true` the entire `scratch_string.this` resource would be replaced!

This allows us to create really tight dependancies on resources that may not be otherwise related in our Terraform code. You should however be very careful using this the referenced resource(`scratch_bool.this`) changes then your resource(`scratch_string.this`) will be replaced.

References _trigger_ replacement in the following conditions:

- If the reference is to a resource with multiple instances, a plan to update or replace any instance will trigger replacement.
- If the reference is to a single resource instance, a plan to update or replace that instance will trigger replacement.
- If the reference is to a single attribute of a resource instance, any change to the attribute value will trigger replacement.