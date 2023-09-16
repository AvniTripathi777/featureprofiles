## Summary
This test checks for DUT functionality of BGP addpath capability. It also confirms accuracy of addpath capabilities negotiation, following checks
1. DUT sets value for the Send/Receive field as "3" during capabilities negotiation to signal support for both Send and Receive ability for multiple paths.
2. DUT sets value for the Send/Receive field as "1" during capabilities negotiation to signal support for Receive ability for multiple paths.
3. DUT sets value for the Send/Receive field as "2" during capabilities negotiation to signal support for Send ability for multiple paths.
4. Tests are conducted at scale of 1M v4 and 600k v6 routes with 64 different NHs each. Also, routes are withdrawn at scale to ensure addpath functionality is maintained at scale. Multipath is also enabled on the DUT to extend addpath functionalities implications on FIB programming at scale. 

## Testbed type

* TESTBED_DUT_ATE_9LINKS

## Topology:


ATE1:Port1 <-EBGP-> DUT:Port1

ATE2:Port2 <-EBGP-> DUT:Port2

ATE3:Port3 <-IBGP-> DUT:Port3

ATE4:Port4 <-IBGP-> DUT:Port4

ATE5:Port5 <-IBGP-> DUT:Port5


### Test procedure
Test-1 and Test-2 below are expected to be run at the global, peer-group and neighbor levels.

**Test-1**: Verify ADDPATH Send/Receive capability
* DUT:Port1 and DUT:Port2 has EBGP peering with ATE1:Port1 and ATE2:Port2 respectively.
* DUT:Port3 and DUT:Port4 has IBGP peering with directly connected ATE3:Port3 and ATE4:Port4 respectively. In this case, DUT is the RR server and ATE3 and ATE4 are RR clients
* DUT:Port5 has IBGP peering with directly connected ATE5:Port5. In this case DUT is the RR client and ATE5:Port5 is the RR server
* Ensure that the EBGP and IBGP peering are enabled for both address famailies.
* DUT should be configured with "Both" Send and Receive ability for addpath on all IBGP peering. Same for ATE3:Port3, ATE4:Port4 and ATE5:Port5
  * During BGP capabilities negotiation in Open message, verify that the DUT negotiated addpath cability with Send/Receive field set to "3"

* Configure ATE1:Port1 and ATE2:Port2 to advertise same prefix "prefix-1" with different Protocol next-hops to DUT:Port1 and DUT:Port2
  * Verify that the DUT is advertising multiple paths to prefix-1 to RRCs ATE3 and ATE4 as well as to the RRS ATE5 with different path-ids
* Configure ATE1:Port1 and ATE2:Port2 to advertise 2 different paths each for prefix-1 with different Protocol next-hops to DUT:Port1 and DUT:Port2 respectively. Then configure DUT to send maximum 3 paths over ADD-Path to ATE3 and ATE4
  * Verify that ATE3 and ATE4 receive only 3 different paths (out of 4) w/ unique path-ids from DUT for prefix-1.
* Configure RRCs ATE3:Port3 and ATE4:Port4 to advertise "prefix-2" to DUT with different Protocol next-hops.
  * Verify that the DUT advertises multiple paths for prefix-2 to ATE5 with different path-ids
* Configure RRCs ATE3:Port3 and ATE4:Port4 each to advertise 2 different paths for "prefix-2" to DUT with different Protocol next-hops each. Then configure DUT to send maximum 3 paths over ADD-Path to ATE5
  * Verify that ATE5 receives only 3 different paths (out of 4) w/ unique path-ids from DUT for prefix-2.

**Test-2**: Verify ADDPATH scaling 
* DUT:Port3 and DUT:Port4 has IBGP peering with directly connected ATE3:Port3 and ATE4:Port4 respectively. In this case, DUT is the RR server and ATE3 and ATE4 are RR clients
* DUT:Port5 has IBGP peering with directly connected ATE5:Port5. In this case DUT is the RR client and ATE5:Port5 is the RR server
* Ensure that the EBGP and IBGP peering are enabled for both address famailies.
* DUT is configued with addpath Receive capability only on its peering with ATE3:Port3 and ATE4:Port4
  * During BGP capabilities negotiation in Open message, verify that the DUT negotiated addpath cability with Send/Receive field set to "1" signaling Receive capability.
 * DUT is configued with addpath Send capability only on its peering with ATE5:Port5
  * During BGP capabilities negotiation in Open message, verify that the DUT negotiated addpath cability with Send/Receive field set to "2" signaling send capability.
 * Enable BGP multipath as well in the DUT.
 * ATE3, 4 and 5 should be conffigured for addpath "both" capability.
 * Configue ATE3 and ATE4 each to advertise 1M v4 prefixes with 32 different NHs for each prefix with a different path-id. Requirement here is for the DUT to receive 1M routes with 64 different NHs for the same prefixes but different path-id. Follow the same process for IPv6 with 600k routes and 32 distict NHs for each prefix from each ATE.
  * Follow the same procedure as above for 300k (out of total 600k) v6 routes.
  * Veirfy on ATE5 that it receives all v4 and v6 routes by checking for the received prefix count. However, for certain set of prefixes (example 500 v4 and 300 v6 prefixes), ensure that they are received with 64 different NHs each with different path-ids.
 * Withdraw 500k v4 prefixes and 300k v6 prefixes from ATE4 and verify the following for the withdrawn prefixes
  * ATE5 now has routes for the withdrawn prefixes only with 32 path-ids originated by ATE3
  * For the withdrawn routes, ensure that the DUT installed the prefixes in its FIB with only 32 NHs generated by ATE3
 
## Config Parameter coverage
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/config/receive
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/config/send
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/config/send-max
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/config/eligible-prefix-policy
* /network-instances/network-instance/protocols/protocol/bgp/global/neighbor/afi-safis/afi-safi/use-multiple-paths/config/enabled
  
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/config/receive
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/config/send
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/config/send-max
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/config/eligible-prefix-policy
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/use-multiple-paths/config/enabled
  
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/config/receive
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/config/send
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/config/send-max
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/config/eligible-prefix-policy
* /network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/afi-safis/afi-safi/use-multiple-paths/config/enabled

## Telemetry Parameter coverage
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/state/receive
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/state/send
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/state/send-max
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/add-paths/state/eligible-prefix-policy
* /network-instances/network-instance/protocols/protocol/bgp/global/afi-safis/afi-safi/use-multiple-paths/state/
  
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/state/receive
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/state/send
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/state/send-max
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/add-paths/state/eligible-prefix-policy
* /network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/afi-safis/afi-safi/use-multiple-paths/state/


* /neighbors/neighbor/afi-safis/afi-safi/add-paths/state
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/state/receive
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/state/send
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/state/send-max
* /neighbors/neighbor/afi-safis/afi-safi/add-paths/state/eligible-prefix-policy
* /network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/supported-capabilities
* /network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/afi-safis/afi-safi/use-multiple-paths/state/
