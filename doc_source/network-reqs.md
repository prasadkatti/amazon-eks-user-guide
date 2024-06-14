--------

 **Help improve this page** 

--------

--------

Want to contribute to this user guide? Scroll to the bottom of this page and select **Edit this page on GitHub**\. Your contributions will help make our user guide better for everyone\.

--------

# Amazon EKS VPC and subnet requirements and considerations<a name="network-reqs"></a>

## VPC requirements and considerations<a name="network-requirements-vpc"></a>

When you create a cluster, the VPC that you specify must meet the following requirements and considerations:
+ The VPC must have a sufficient number of IP addresses available for the cluster, any nodes, and other Kubernetes resources that you want to create\. If the VPC that you want to use doesn’t have a sufficient number of IP addresses, try to increase the number of available IP addresses\.

  You can do this by updating the cluster configuration to change which subnets and security groups the cluster uses\. You can update from the AWS Management Console, the latest version of the AWS CLI, AWS CloudFormation, and `eksctl` version `v0.164.0-rc.0` or later\. You might need to do this to provide subnets with more available IP addresses to successfully upgrade a cluster version\.
**Important**  
All subnets that you add must be in the same set of AZs as originally provided when you created the cluster\. New subnets must satisfy all of the other requirements, for example they must have sufficient IP addresses\.

For example, assume that you made a cluster and specified four subnets\. In the order that you specified them, the first subnet is in the `us-west-2a` Availability Zone, the second and third subnets are in `us-west-2b` Availability Zone, and the fourth subnet is in `us-west-2c` Availability Zone\. If you want to change the subnets, you must provide at least one subnet in each of the three Availability Zones, and the subnets must be in the same VPC as the original subnets\.

\+

\+

If you need more IP addresses than the CIDR blocks in the VPC have, you can add additional CIDR blocks by [associating additional Classless Inter\-Domain Routing \(CIDR\) blocks](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#add-ipv4-cidr) with your VPC\. You can associate private \(RFC 1918\) and public \(non\-RFC 1918\) CIDR blocks to your VPC either before or after you create your cluster\. It can take a cluster up to five hours for a CIDR block that you associated with a VPC to be recognized\.

\+

You can conserve IP address utilization by using a transit gateway with a shared services VPC\. For more information, see [Isolated VPCs with shared services](https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-isolated-shared.html) and [Amazon EKS VPC routable IP address conservation patterns in a hybrid network](https://aws.amazon.com/blogs/containers/eks-vpc-routable-ip-address-conservation/)\. \* If you want Kubernetes to assign `IPv6` addresses to Pods and services, associate an `IPv6` CIDR block with your VPC\. For more information, see [Associate an IPv6 CIDR block with your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#vpc-associate-ipv6-cidr) in the Amazon VPC User Guide\. \* The VPC must have `DNS` hostname and `DNS` resolution support\. Otherwise, nodes can’t register to your cluster\. For more information, see [DNS attributes for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html) in the Amazon VPC User Guide\.

If you created a cluster with Kubernetes `1.14` or earlier, Amazon EKS added the following tag to your VPC:

This tag was only used by Amazon EKS\. You can remove the tag without impacting your services\. It’s not used with clusters that are version `1.15` or later\.

## Subnet requirements and considerations<a name="network-requirements-subnets"></a>

When you create a cluster, Amazon EKS creates 2–4 [https://docs\.aws\.amazon\.com/](https://docs.aws.amazon.com/) AWSEC2/latest/UserGuide/using\-eni\.html\[elastic network interfaces\] in the subnets that you specify\. These network interfaces enable communication between your cluster and your VPC\. These network interfaces also enable Kubernetes features such as `kubectl exec` and `kubectl logs`\. Each Amazon EKS created network interface has the text `Amazon EKS cluster-name ` in its description\.

Amazon EKS can create its network interfaces in any subnet that you specify when you create a cluster\. You can change which subnets Amazon EKS creates its network interfaces in after your cluster is created\. When you update the Kubernetes version of a cluster, Amazon EKS deletes the original network interfaces that it created, and creates new network interfaces\. These network interfaces might be created in the same subnets as the original network interfaces or in different subnets than the original network interfaces\. To control which subnets network interfaces are created in, you can limit the number of subnets you specify to only two when you create a cluster or update the subnets after creating the cluster\.

### Subnet requirements for clusters<a name="cluster-subnets"></a>

The [subnets](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html#subnet-types) that you specify when you create or update a cluster must meet the following requirements:
+ The subnets must each have at least six IP addresses for use by Amazon EKS\. However, we recommend at least 16 IP addresses\.
+ The subnets can be a public or private\. However, we recommend that you specify private subnets, if possible\. A public subnet is a subnet with a route table that includes a route to an [internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html), whereas a private subnet is a subnet with a route table that doesn’t include a route to an internet gateway\.
+ The subnets can’t reside in the following Availability Zones:

### IP address family usage by component<a name="network-requirements-ip-table"></a>

The following table contains the IP address family used by each component of Amazon EKS\. You can use a network address translation \(NAT\) or other compatibility system to connect to these components from source IP addresses in families with the "No" value for a table entry\.

Functionality can differ depending on the IP family \(`ipFamily`\) setting of the cluster\. This setting changes the type of IP addresses used for the CIDR block that Kubernetes assigns to Services\. A cluster with the setting value of IPv4 is referred to as an  *IPv4 cluster* , and a cluster with the setting value of IPv6 is referred to as an  *IPv6 cluster* \.

**Note**  
  ^1^ The endpoint is dual stack with both `IPv4` and `IPv6` addresses\. Your applications outside of AWS, your nodes for the cluster, and your pods inside the cluster can reach this endpoint by either `IPv4` or `IPv6`\.  
  ^2^ You choose between an `IPv4` cluster and `IPv6` cluster in the IP family \(`ipFamily`\) setting of the cluster when you create a cluster and this can’t be changed\. Instead, you must choose a different setting when you create another cluster and migrate your workloads\.

### Subnet requirements for nodes<a name="node-subnet-reqs"></a>

You can deploy nodes and Kubernetes resources to the same subnets that you specify when you create your cluster\. However, this isn’t necessary\. This is because you can also deploy nodes and Kubernetes resources to subnets that you didn’t specify when you created the cluster\. If you deploy nodes to different subnets, Amazon EKS doesn’t create cluster network interfaces in those subnets\. Any subnet that you deploy nodes and Kubernetes resources to must meet the following requirements:
+ The subnets must have enough available IP addresses to deploy all of your nodes and Kubernetes resources to\.
+ If you want Kubernetes to assign `IPv6` addresses to Pods and services, then you must have one `IPv6` CIDR block and one `IPv4` CIDR block that are associated with your subnet\. For more information, see [Associate an IPv6 CIDR block with your subnet](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-subnets.html#subnet-associate-ipv6-cidr) in the Amazon VPC User Guide\. The route tables that are associated with the subnets must include routes to `IPv4` and `IPv6` addresses\. For more information, see [Routes](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html#route-table-routes) in the Amazon VPC User Guide\. Pods are assigned only an `IPv6` address\. However the network interfaces that Amazon EKS creates for your cluster and your nodes are assigned an `IPv4` and an `IPv6` address\.
+ If you need inbound access from the internet to your Pods, make sure to have at least one public subnet with enough available IP addresses to deploy load balancers and ingresses to\. You can deploy load balancers to public subnets\. Load balancers can load balance to Pods in private or public subnets\. We recommend deploying your nodes to private subnets, if possible\.
+ If you want to deploy load balancers to a subnet, the subnet must have the following tag:
  + Private subnets
+ Public subnets

When a Kubernetes cluster that’s version `1.18` and earlier was created, Amazon EKS added the following tag to all of the subnets that were specified\.

If you deployed a VPC by using `eksctl` or any of the Amazon EKS AWS CloudFormation VPC templates, the following applies:
+  **On or after March 26, 2020** – Public `IPv4` addresses are automatically assigned by public subnets to new nodes that are deployed to public subnets\.
+  **Before March 26, 2020** – Public `IPv4` addresses aren’t automatically assigned by public subnets to new nodes that are deployed to public subnets\.

This change impacts new node groups that are deployed to public subnets in the following ways:

## Shared subnet requirements and considerations<a name="network-requirements-shared"></a>

You can use *VPC sharing* to share subnets with other AWS accounts within the same AWS Organizations\. You can create Amazon EKS clusters in shared subnets, with the following considerations:
+ The owner of the VPC subnet must share a subnet with a participant account before that account can create an Amazon EKS cluster in it\.
+ You can’t launch resources using the default security group for the VPC because it belongs to the owner\. Additionally, participants can’t launch resources using security groups that are owned by other participants or the owner\.
+ In a shared subnet, the participant and the owner separately controls the security groups within each respective account\. The subnet owner can see security groups that are created by the participants but cannot perform any actions on them\. If the subnet owner wants to remove or modify these security groups, the participant that created the security group must take the action\.
+ If a cluster is created by a participant, the following considerations apply:
  + All nodes must be made by the same participant, including managed node groups\.
+ The shared VPC owner cannot view, update or delete a cluster that a participant creates in the shared subnet\. This is in addition to the VPC resources that each account has different access to\. For more information, see [Responsibilities and permissions for owners and participants](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html#vpc-share-limitations) in the *Amazon VPC User Guide*\.

For more information about VPC subnet sharing, see [Share your VPC with other accounts](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html#vpc-share-limitations) in the *Amazon VPC User Guide*\.