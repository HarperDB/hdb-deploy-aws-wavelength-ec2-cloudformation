# HarperDB on AWS Wavelength 
CloudFormation templates for building HarperDB 3.1.3 nodes on [AWS Wavelength](https://aws.amazon.com/wavelength/).

## Usage
1. Create a VPC with a Carrier Gateway and a Wavelength Zone subnet.
    * This template can be used multiple times, once for each Wavelength Zone that will receive HarperDB node(s).
```bash
aws --region "us-west-2" cloudformation create-stack \
    --template-body "file://cloudformation-templates/harperdb-wavelength-network.yaml" \
    --stack-name "harperdb-wl-network-den" \
    --parameters \
        ParameterKey=VPCCidrBlock,ParameterValue="10.1.0.0/16" \
        ParameterKey=AvailabilityZone,ParameterValue="us-west-2-wl1-den-wlz-1" \
        ParameterKey=SubnetCidrBlock,ParameterValue="10.1.0.0/24"
```

2. Create a HarperDB node in a Wavelength Zone.
    * Each HarperDB node consists of an EC2 instance running HarperDB, with a TLS certificate from Let's Encrypt, and Nginx 
  forwarding requests to either the HarperDB API or HarperDB Custom Functions.
    * This template, along with the harperdb-wavelength-dns.yaml template, can be used multiple times to create multiple HarperDB nodes in the same or different Wavelength Zones.
```bash
aws --region "us-west-2" cloudformation create-stack \
    --template-body "file://cloudformation-templates/harperdb-wavelength-instance.yaml" \
    --stack-name "harperdb-wl-instance-den01" \
    --parameters \
        ParameterKey=VPC,ParameterValue="<VPC ID>" \
        ParameterKey=AvailabilityZone,ParameterValue="us-west-2-wl1-den-wlz-1" \
        ParameterKey=Subnet,ParameterValue="<Subnet ID>" \
        ParameterKey=InstanceType,ParameterValue="t3.medium" \
        ParameterKey=KeyName,ParameterValue="<Key Name>" \
        ParameterKey=RootVolumeSize,ParameterValue="8" \
        ParameterKey=RootVolumeType,ParameterValue="gp2" \
        ParameterKey=DataVolumeSize,ParameterValue="10" \
        ParameterKey=DataVolumeType,ParameterValue="gp2" \
        ParameterKey=Fqdn,ParameterValue="den01.yourdomain.io" \
        ParameterKey=FqdnCf,ParameterValue="functions-den01.yourdomain.io" \
        ParameterKey=HdbAdminUsername,ParameterValue="HDB_ADMIN" \
        ParameterKey=HdbAdminPassword,ParameterValue="abc123!" \
        ParameterKey=IamInstanceProfile,ParameterValue="<IAM Instance Profile Name>"

```

3. Create two Route53 DNS records for each HarperDB node. One for the HarperDB API and another for HarperDB Custom Functions.
    * This template, along with the harperdb-wavelength-instance.yaml template, can be used multiple times to create multiple HarperDB nodes in the same or different Wavelength Zones.
```bash
aws --region "us-west-2" cloudformation create-stack \
    --template-body "file://cloudformation-templates/harperdb-wavelength-dns.yaml" \
    --stack-name "harperdb-wl-dns-den01" \
    --parameters \
        ParameterKey=HostedZoneId,ParameterValue="<Route53 Hosted Zone ID>" \
        ParameterKey=Fqdn,ParameterValue="den01.yourdomain.io" \
        ParameterKey=FqdnCf,ParameterValue="functions-den01.yourdomain.io" \
        ParameterKey=CarrierIP,ParameterValue="<Carrier IP address assigned to instance>"
```

4. Add the HarperDB node(s) to [HarperDB Studio](https://studio.harperdb.io/sign-up) as User-Installed Instances
    * HarperDB Studio is a UI for easy management of HarperDB nodes.
    * [HarperDB Studio docs](https://harperdb.io/docs/harperdb-studio/)