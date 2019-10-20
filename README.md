# Kubernetes-V: Vanilla Kubernetes v1.16.1 with Kubeadm deployed with Terraform on AWS

## Introduction

This repo is a merge of these 2 repos: 

[https://github.com/terraform-aws-modules/terraform-aws-vpc](https://github.com/terraform-aws-modules/terraform-aws-vpc)

[https://github.com/scholzj/terraform-aws-kubernetes](https://github.com/scholzj/terraform-aws-kubernetes)

with some minor adpations to get k8s 1.16.1 with Kubeadm deployed in a new VPC on AWS with docker, containerd or cri-o.

In the first step we'll be using docker as container runtime to create our cluster with 1 master and 3 nodes.

## Prerequisits

Terraform v0.12.12

## Installation with docker

### Create a new kubernetes-v VPC 

```bash
git clone https://github.com/arashkaffamanesh/kubernetes-v.git
cd kubernetes-v/terraform-aws-vpc/examples/simple-vpc
terraform init
terraform plan
terraform apply
```

 Provide the private subnets from the vpc output creation above for the worker subnets and one of the public subnets for the master subnet in main.tf under `terraform-aws-kubernetes/examples/` and adapt the hosted zone domain name similar to this:

```bash
cd ../../../terraform-aws-kubernetes/examples/

vim main.tf

  hosted_zone          = "kubernauts-v.de"
  hosted_zone_private  = false
  master_subnet_id = "subnet-0b38e00b1aaf1a8f5"
  worker_subnet_ids = [
    "subnet-0ec3ce6d94c0df08d",
    "subnet-02f4bed988e132366",
    "subnet-07af7944681a6162c",
  ]

terraform init
terraform plan
terraform apply
```

You should now have 1 master and 3 worker nodes and would need to grab the admin.conf from the master node under /etc/kubernetes/ to your local machine and provide the public IP address of the master in admin.conf:

```bash
ssh centos@<master node ip>
sudo cat /etc/kubernetes/admin.conf
exit
vim admin.conf
# paste the content of admin.conf and adjust the IP address of the master node and run:
KUBECONFIG=admin.conf kubectl get nodes
```

You should get something like this:

```bash
examples $ KUBECONFIG=admin.conf kubectl get nodes
NAME                                            STATUS   ROLES    AGE   VERSION
ip-10-0-1-106.eu-central-1.compute.internal     Ready    <none>   10m   v1.16.1
ip-10-0-101-117.eu-central-1.compute.internal   Ready    master   11m   v1.16.1
ip-10-0-2-173.eu-central-1.compute.internal     Ready    <none>   10m   v1.16.1
ip-10-0-3-164.eu-central-1.compute.internal     Ready    <none>   10m   v1.16.1
```

To label the nodes properly, you might want to run something like this:

```bash
KUBECONFIG=admin.conf kubectl label node ip-10-0-1-106.eu-central-1.compute.internal node-role.kubernetes.io/node=
```

## Clean up

Run `terraform destroy` from the directories where you deployed k8s and the vpc.

## Installation with containerd

In this step we'll clean up our machines by resetting kubeadm and removing docker and the iptable rules on all machines as follow:

```bash
sudo kubeadm reset
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -F
sudo iptables -X
sudo yum remove docker docker-common docker-selinux docker-engine 
sudo rm -rf /var/lib/docker
```

Note B.: you can ssh into the worker nodes from the master by providing your id_rsa key on the master!


... work in process

## Credits

Thanks givings goes to the awesome [Jakub Scholz](https://twitter.com/scholzj) and the contributors of [Terraform AWS VPC implementation](https://github.com/terraform-aws-modules/terraform-aws-vpc/graphs/contributors).

## Related links

[Container Runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

[Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-control-plane-node)

[Calico](https://docs.projectcalico.org/v3.10/introduction/)


