Currently the restoration process implies taking manual action directly on the machines that want to be restored. ReaR generates an ISO image that the machine that wants to be restored must boot from.

The method recommended to restore is to connect that ISO directly to either the VMs or the bare metal servers, which prevents it from being automated. However, we can leverage the capacities of ReaR itself to automate the restores. ReaR supports doing backups directly to a PXE server. Once the machines that we want to restore are configured to use that PXE server to boot, they could directly boot to the ReaR ISO with the restoration script.

This is very helpful, as it would allow the operator to order restorations interactively without having to do any manual work.

The best part is that a PXE server is already available for Openstack nodes, and all of them are already configured to use it. The PXE server is embedded in the Ironic component of Openstack. The idea within this RFE is to use the PXE server offered by Ironic and used by all the Openstack nodes to install their OS. ReaR will export the ISOs of the backups in a PXE acceptable format using the PXE related configuration parameters as stated in:

https://raw.githubusercontent.com/rear/rear/master/usr/share/rear/conf/examples/PXE-booting-example-with-URL-style.conf

This RFE should be paired with an ansible role for automating restoration on kvm virtual machines.
