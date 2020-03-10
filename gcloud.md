gcloud config set project gns3-235219
=======
VPC
=======

gcloud compute networks create west2 \
    --subnet-mode=custom \
    --region=us-west1

gcloud compute networks subnets create sub12 \
    --network=west2 \
    --region=us-west1 \
    --range=10.1.12.0/24

gcloud compute networks subnets list --network=west1

=========
Firewall
=========

gcloud compute firewall-rules create <FIREWALL_NAME> \
    --network=west2 \
    --allow tcp,udp,icmp\
    --source-ranges <IP_RANGE>

gcloud compute firewall-rules create <FIREWALL_NAME> \
    --network west2 --allow tcp:22,tcp:3389,icmp

gcloud compute firewall-rules create sshrule2 \
    --direction=INGRESS \
    --priority=1000 \
    --network=west1 \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=netops \

gcloud compute --project=gns3-235219 firewall-rules create test \
    --direction=INGRESS \
    --priority=1000 \
    --network=west1 \
    --action=ALLOW \
    --rules=all \
    --source-ranges=10.11.11.0/24

gcloud compute firewall-rules create openvpn1 \
    --direction=INGRESS \
    --priority=1000 \
    --network=west1 \
    --action=ALLOW \
    --rules=udp:1194 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=vpn

gcloud compute --project=gns3-235219 firewall-rules create allow-vpn2 --direction=INGRESS --priority=1000 --network=west2 --action=ALLOW --rules=udp:1194 --source-ranges=0.0.0.0/0 --target-tags=vpn

gcloud compute firewall-rules create my-network-allow-ssh \
--direction=INGRESS \
--priority=1000 \
--network=my-network \
--action=ALLOW \
--rules=tcp:22 \
--source-ranges=0.0.0.0/0

gcloud compute firewall-rules create my-network-allow-internal \
    --direction=INGRESS \
    --priority=1000 \
    --network=my-network \
    --action=ALLOW \
    --rules=all \
    --source-ranges=192.168.1.0/24,192.168.2.0/24
=====================
static external IP addresses
=====================
gcloud compute addresses create east-address-1 --region=us-east1

=================
instance
=================
Create a virtual machine to act as a NAT gateway on my-network:

gcloud compute instances create nat-gateway --network my-network \
    --subnet subnet-us-central1 \
    --can-ip-forward \
    --zone us-central1-a 

Create a new virtual machine without an external IP address:

gcloud compute instances create private-instance \
    --network my-network \
    --subnet subnet-us-central1 \
    --no-address \
    --zone us-central1-a \
    --tags no-ip

=============
routes
============
Create a route to send traffic destined to the internet through your gateway instance:

gcloud compute routes create nat-route \
    --network my-network \
    --destination-range 0.0.0.0/0 \
    --next-hop-instance nat-gateway \
    --next-hop-instance-zone us-central1-a \
    --tags no-ip --priority 800


delete --quit