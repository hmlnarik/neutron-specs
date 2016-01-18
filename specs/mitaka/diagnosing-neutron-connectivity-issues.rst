..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================
Diagnosing Neutron connectivity issues
======================================

Include the URL of your launchpad RFE:

**TODO** https://bugs.launchpad.net/neutron/+bug/example-id

This spec describes extension of neutron-debug CLI with commands that ease
debugging some types of networking issues by detailing diagnostics.


Problem Description
===================
Neutron-debug currently supports commands related to probe port manipulation
and a ping-all command. [CMDS]_ These commands do not however bring much
information on the validity of actual network setup.

Proposed Change
===============
This spec suggest enhancing neutron-debug with a set of commands
that offer more fine-grained commands for diagnostic of networking
setup. They are primarily aimed to answer the question
"Why cannot I ping my floating IP?" and expect operating either
from network or compute node (depending on the context of the command).

Each of the commands outputs a list of tests and whether they passes/failed.
The list of commands and the tests follows:

* Diagnose validity of security groups for ping/TCP/UDP traffic

  | Command: ``diagnose-secgroups-ping [ <router-id> | <port-id> | <instance-id>]``
  | Command: ``diagnose-secgroups-tcp --tcp-port <NN> [ <router-id> | <port-id> | <instance-id>]``
  | Command: ``diagnose-secgroups-udp --udp-port <NN> [ <router-id> | <port-id> | <instance-id>]``

  Should be run from: any node

  This command uses API calls to check whether security groups allow external
  traffic to go trough to the given port.

* Diagnose validity of router settings and verify that target IP can be pinged
  directly from the router. Target-IP can be both private and floating.

  Command: ``diagnose-router <router-id> <target-IP>``

  Should be run from: router node

  This command depends on actual networking implementation, here Linux/OVS
  implementation is used for illustration of the tests.

  Tests:

  * Check if namespace is configured on L3 agent host
  * Ping the *target-IP* from DHCP namespace
  * Ping the *target-IP* from router namespace
  * Check that floating IP is assigned at router [only when target-IP is floating]
  * Check that both GW port and port facing LAN are in enabled admin_state_up

Initial release will only support Linux hosts and OVS switches.

The tests in the inital stage will currently only consist of unit tests
similarly to the existing neutron-debug code.

Future work
===========
The weakness of neutron-debug command is that it cannot be executed remotely.
After completing this spec, the next step would be to transform the current
single-node CLI that executes local commands into a CLI and agent. CLI would
then be able to invoke commands remotely via RPC, thus eliminating needs to
get access to the affected system beforehand.

This architecture would also create a basis upon which GUI diagnostic tools
can operate. Demand for this tool can be seen e.g. in
`bug 1507499 <https://bugs.launchpad.net/neutron/+bug/1507499>`_.

References
==========

.. [CMDS] http://docs.openstack.org/cli-reference/content/neutron-debug_commands.html
