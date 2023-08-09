# Data Sources
_Data sources_ allow Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions. Each [provider](https://developer.hashicorp.com/terraform/language/providers) may offer data sources alongside its set of [resource](https://developer.hashicorp.com/terraform/language/resources) types.

## Using Data Sources

A data source is accessed via a special kind of resource known as a _data resource_, declared using a `data` block:

```hcl
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```

- A `data` block requests that Terraform read from a given data source ("aws_ami") and export the result under the given local name ("example"). The name is used to refer to this resource from elsewhere in the same Terraform module, but has no significance outside of the scope of a module.
- The data source and name together serve as an identifier for a given resource and so must be unique within a module.
- Within the block body (between `{` and `}`) are query constraints defined by the data source. Most arguments in this section depend on the data source, and indeed in this example `most_recent`, `owners` and `tags` are all arguments defined specifically for [the `aws_ami` data source](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ami).
- When distinguishing from data resources, the primary kind of resource (as declared by a `resource` block) is known as a _managed resource_. Both kinds of resources take arguments and export attributes for use in configuration, but while managed resources cause Terraform to create, update, and delete infrastructure objects, data resources cause Terraform only to _read_ objects. For brevity, managed resources are often referred to just as "resources" when the meaning is clear from context.


## Difference between Resource and Data Source?
| Resource                                           | Data Source                  |
| -------------------------------------------------- | ---------------------------- |
| Keyword: `resource`                                | Keyword: `data`              |
| `Creates`, `Updates` and `Destroys` Infrastructure | Only reads Infrastructure    |
| Also called `Managed Resources`                    | Also called `Data Resources` |                                                   |                              |
