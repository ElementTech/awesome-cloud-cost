<div align="center">

<!-- title -->

<!--lint ignore no-dead-urls-->

# Awesome Cloud Cost
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
<!-- [![lint](https://github.com/jatalocks/awesome-cloud-cost/actions/workflows/lint.yaml/badge.svg)](https://github.com/jatalocks/awesome-cloud-cost/actions/workflows/lint.yaml) -->

<!-- subtitle -->

A curated list of awesome tips, tricks and hacks for saving cost on the cloud

<!-- image -->
<!-- 
<a href="" target="_blank" rel="noopener noreferrer">
  <img src="" />
</a> -->

<!-- description -->

</div>

<!-- TOC -->

## Contents

- [Awesome Cloud Cost](#awesome-cloud-cost)
  - [Contents](#contents)
  - [General](#general)
  - [AWS](#aws)
    - [Compute](#compute)
    - [Networking](#networking)
    - [S3](#s3)
  - [Kubernetes](#kubernetes)
  - [Contributing](#contributing)
  - [Contributors](#contributors)

<!-- CONTENT -->
## General

- Cache external dependencies locally to reduce network transfer costs. For example - pull-through docker image registries.
  - [Harbor](https://goharbor.io/)
    - [Kubernetes Mutating Webhook](https://github.com/indeedeng-alpha/harbor-container-webhook)

- Always prefer spot instances where stateless and possible as opposed to on-demand.

- Use automated scaling solutions to power off dev workloads during weekends if possible.

- Filter your logs, metrics and traces before they reach your monitoring solution. In almost all solutions, SaaS or not, you're being charged for their storage or ingestion.

## AWS

### Compute

- Use Reserved Instances and Savings Plans. Consider using "smart" automated RI SaaS solutions which are based on your existing workloads.
  - [spot.io](https://spot.io/)
  - [zesty.co](https://zesty.co/)

- Prefer higher generation EC2 instances, they will always be cheaper. It is also true for other products such as storage solutions like gp2 as opposed to gp3.
  - [EC2 Types List](https://instances.vantage.sh/)

### Networking

- Move away from Classic load balancers as they are [deprecated](https://aws.amazon.com/blogs/aws/ec2-classic-is-retiring-heres-how-to-prepare/) for EC2-Classic networks and cost more, use Network or Application load balancers instead.

- Most likely, move away from VPC peering to [Transit Gateways](https://aws.amazon.com/transit-gateway/) (or [Network Manager](https://aws.amazon.com/transit-gateway/network-manager/)) and [VPC Sharing](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html). Peering is costlier when there are many VPCs. Take the time to calculate your usage and network traffic.

- Use [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html) when your workloads access AWS services from within AWS. Check however if the cost of the endpoint isn't higher than the cost of your usage without it.

- When using multiple private subnets that access the internet, make sure they each have a NAT gateway. It will usually cost more to send the traffic only through one of them on a heavy networking load. Alternatively, setup a NAT on a small instance yourself, or use only public subnets and block all external access - whichever fits your budget and use-case the best.
  - <details>
    <summary>Self Hosted NAT Gateway Tip</summary>
    <p>If you're on a shoestring budget and internet access from your private subnets doesn't absolutely require 100% uptime, you can use a `t3a.nano` as a NAT instance instead of using NAT gateways, which are quite expensive per-subnet-month.</p>

### S3

- Use [S3 object classes](https://aws.amazon.com/s3/storage-classes/) to majorly reduce costs on less frequently accessed buckets.

- If using S3 Glacier, [compress your files](https://aws.amazon.com/blogs/storage/compressing-and-archiving-logs-to-the-amazon-s3-glacier-storage-classes/) into as few as possible before uploading in order to save requests cost.

## Kubernetes

- Consolidate your pods on less nodes. Leave only as little headroom as you intend for in your nodes.
  - [Cluster-Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)
  - [AWS-Karpenter](https://karpenter.sh/)

- Don't over commit resources. Pod requests must be optimized over time in order to not over provision.
  - [ScaleOps](https://www.scaleops.co/)
  - [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)

- If possible, prefer using only a single region to avoid network transfer costs between nodes. Preferably when it's not production.
  - <details>
      <summary>Single Region Highly Available AWS EKS Karpenter Trick</summary>
      <p>This dual-provisioner configuration allows Karpenter to softly always prefer scheduling on a single AZ, unless it is unavailable. In this scenario, it will move to another AZ until the former AZ works.</p>

      ```yaml
      # Provisioner A
      - providerName: main-node
        weight: 100
        requirements:
          - key: "topology.kubernetes.io/zone"
            operator: In
            values: ["us-east-1a", "eu-west-1a"]
      ---
      # Provisioner B
      - providerName: backup-node
        weight: 0
        requirements:
          - key: "topology.kubernetes.io/zone"
            operator: NotIn
            values: ["us-east-1a", "eu-west-1a"]
      ```

      </details>

<!-- END CONTENT -->

## Contributing

[Contributions of any kind welcome, just follow the guidelines](contributing.md)!

## Contributors

[Thanks goes to these contributors](https://github.com/jatalocks/awesome-cloud-cost/graphs/contributors)!
