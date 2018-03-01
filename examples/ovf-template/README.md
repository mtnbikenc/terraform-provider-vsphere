# vSphere OVF Template Deploy Example

This example is designed to demonstrate the ability of the [Terraform vSphere
Provider][ref-tf-vsphere] to work with OVF templates, such as the [VMware
images][ref-coreos-vsphere] offered by [CoreOS Container
Linux][ref-coreos-linux].

[ref-tf-vsphere]: https://www.terraform.io/docs/providers/vsphere/index.html
[ref-coreos-vsphere]: https://coreos.com/os/docs/latest/booting-on-vmware.html
[ref-coreos-linux]: https://coreos.com/os/docs/latest/

This repository requires version 1.3.0 or higher of the Terraform vSphere
Provider, in addition to [govc][ref-govc] to allow for upload of the template.

[ref-govc]: https://github.com/vmware/govmomi/tree/master/govc

This example performs the following:

* Downloads and builds a demonstration "web application", which is mainly just a
  HTTP server that serves up a hello world application on port 8080. This has
  been sourced from [http-hello-go][ref-http-hello-go].
* The `Makefile` and repository also has scripts to download the latest CoreOS
  VMware OVA, and import it into vSphere. Any previously installed templates are
  archived.
* Finally, the Terraform files in the [`terraform`](terraform/) directory
  contain all that is is needed to deploy the application to a set of virtual
  machines in 3-node vSphere cluster. Ignition (with configuration generated by
  the [Terraform Ignition Provider][ref-tf-provider-ignition]) bootstraps the
  servers with network information, administration SSH keys, and a temporary
  provisioning key managed by the [Terraform TLS Provider][ref-tf-provider-tls].
  [Provisioners][ref-terraform-provisioners] take care of uploading the service
  binary, starting the service, and any extra cleanup steps.

[ref-http-hello-go]: https://github.com/vancluever/http-hello-go
[ref-tf-provider-ignition]: https://www.terraform.io/docs/providers/ignition/index.html
[ref-tf-provider-tls]: https://www.terraform.io/docs/providers/tls/index.html
[ref-terraform-provisioners]: https://www.terraform.io/docs/provisioners/index.html

This repository particularly utilizes the vApp property management capabilities
of the [`vsphere_virtual_machine` resource][ref-tf-vsphere-vm]. For more
details, see the section on [Using vApp properties to supply OVF/OVA
configuration][ref-tf-vsphere-vm-vapp].

[ref-tf-vsphere-vm]: https://www.terraform.io/docs/providers/vsphere/r/virtual_machine.html
[ref-tf-vsphere-vm-vapp]: https://www.terraform.io/docs/providers/vsphere/r/virtual_machine.html#using-vapp-properties-to-supply-ovf-ova-configuration

## Requirements

* A working vCenter installation. This example does not support ESXi. You must
  have at least 3 hosts in a single cluster.
* A virtual network and datastore to deploy the virtual machines in.

## Usage Details

To keep the example as easy to use as possible, you should probably just clone
the entire [terraform-provider-vsphere][ref-tf-vsphere-github] repository. This
will ensure that the application is correctly in your GOPATH (given that is
where you clone the project) and will be built properly. Once done, edit the
`terraform.tfvars.example` file, populating the fields with the relevant values,
and then rename it to `terraform.tfvars`. 

You will also need to supply the correct environment variables for both
Terraform and govc. To ease the process in the example, there is a
`preload_vars.sh` script in the [`scripts`](scripts/) directory, which can be
run with:

```
. scripts/preload_vars.sh
```

[ref-tf-vsphere-github]: https://github.com/terraform-providers/terraform-provider-vsphere

Once done, run:

* `make build` to build the binary
* `make template` to download a copy of the CoreOS OVA to your vSphre cluster
* `make infrastructure` to deploy the virtual machines.

Once deployed, you should be able to check each of the VMs that were deployed on
their respective IP addresses (ie: 10.0.0.100-10.0.0.102) on port 8080 and see
the application that was deployed. Also, if you populated `management_ssh_keys`,
you should be able to SSH into each machine as the `core` user to verify what
has been done!

When you are finished and you want to tear down the virtual machines, run:

```
TF_CMD=destroy make infrastructure
```