### What is Packer?
Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image.

A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. Some examples include AMIs for EC2, VMDK/VMX files for VMware, OVF exports for VirtualBox, etc.

1. Images as code

# Packer Terminology

There are a handful of terms used throughout the Packer documentation where the meaning may not be immediately obvious if you haven't used Packer before. Luckily, there are relatively few. This page documents all the terminology required to understand and use Packer. The terminology is in alphabetical order for quick referencing.

- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Artifacts`](https://developer.hashicorp.com/packer/docs/terminology#artifacts) are the results of a single build, and are usually a set of IDs or files to represent a machine image. Every builder produces a single artifact. As an example, in the case of the Amazon EC2 builder, the artifact is a set of AMI IDs (one per region). For the VMware builder, the artifact is a directory of files comprising the created virtual machine.
    
- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Builds`](https://developer.hashicorp.com/packer/docs/terminology#builds) are a single task that eventually produces an image for a single platform. Multiple builds run in parallel. Example usage in a sentence: "The Packer build produced an AMI to run our web application." Or: "Packer is running the builds now for VMware, AWS, and VirtualBox."
    
- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Builders`](https://developer.hashicorp.com/packer/docs/terminology#builders) are components of Packer that are able to create a machine image for a single platform. Builders read in some configuration and use that to run and generate a machine image. A builder is invoked as part of a build in order to create the actual resulting images. Example builders include VirtualBox, VMware, and Amazon EC2.
    
- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Commands`](https://developer.hashicorp.com/packer/docs/terminology#commands) are sub-commands for the `packer` program that perform some job. An example command is "build", which is invoked as `packer build`. Packer ships with a set of commands out of the box in order to define its command-line interface.
    
- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Data Sources`](https://developer.hashicorp.com/packer/docs/terminology#data-sources) are components of Packer that fetch data from outside Packer and make it available to use within the template. Example of data sources include Amazon AMI, and Amazon Secrets Manager.
    
- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Post-processors`](https://developer.hashicorp.com/packer/docs/terminology#post-processors) are components of Packer that take the result of a builder or another post-processor and process that to create a new artifact. Examples of post-processors are compress to compress artifacts, upload to upload artifacts, etc.
    
- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Provisioners`](https://developer.hashicorp.com/packer/docs/terminology#provisioners) are components of Packer that install and configure software within a running machine prior to that machine being turned into a static image. They perform the major work of making the image contain useful software. Example provisioners include shell scripts, Chef, Puppet, etc.
    
- [](https://developer.hashicorp.com/packer/docs/terminology#)
    
    [`Templates`](https://developer.hashicorp.com/packer/docs/terminology#templates) are either [HCL](https://packer.io/templates/hcl_templates) or JSON files which define one or more builds by configuring the various components of Packer. Packer is able to read a template and use that information to create multiple machine images in parallel.