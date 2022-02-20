# VMify
**From Docker Image to the Cloud in Seconds**

## Why

Infrastructure has gotten needlessly complicated. Traditional container deployment solutions like Kubernetes or ECS
are essentially containers with their own provisioning, scaling and networking running on a pool of VMs with their own
provisioning, scaling and networking. This is a legacy layer of indirection that has grown out of the fact that VMs
have traditionally been much harder to provision than containers.

With VMify this indirection can be eliminated and infrastructure can be made simpler, more reliable, more secure and
cheaper at the same time.

## What

VMify takes your Docker images and turns them into minimal, secure and fully baked AWS AMIs in seconds. 

All it takes is one simple command:

`$ vmify hello-world`
<pre>
$ vmify hello-world
  <span color="#888">0.012</span> <b>🚀 VMify 0.0.1 Personal Edition</b>
  0.089 Pulling hello-world (linux/amd64) ...
  3.987 Squashing layers ...
  3.988 Extracting metadata ...
  4.952 Converting to machine image ...
  5.696 Creating EBS snapshot in eu-central-1 ...
  6.276 Uploading snap-051c47b83d38e84fa (32.1 MiB)...
 11.610 Registering AMI ...
 11.875 ami-075aa8250f1fbe33f successfully created
</pre>
![Output](screenshot.png?raw=true)

You can then integrate this AMI is your usual **infrastructure as code** deployment processes using CloudFormation and Terraform.
Or you can simply launch instances based on it using AWS AutoScaling Groups, the EC2 RunInstances API or the AWS Console.

## How

### Simple
VMify effectively compiles a machine image by combining your Docker image and VMify NanoOS, enabling it to boot directly
on virtual hardware such as AWS EC2.

### Minimal
VMify NanoOS consists just a Linux kernel and a minimal in-memory init system weighing just around 1 MB. All it does is
load the required drivers for the current machine, set up an ACPI daemon to react to reboot and poweroff events and
enable clock synchronization using an NTP daemon to prevent clock drift. After that it passes control to your container
image by loading it from a read-only disk partition and launching its entrypoint and cmd in a confined chroot
environment.

### Fully baked
There is no runtime provisioning and no Docker daemon as the image is already fully backed. Instances boot almost
instantly and are guaranteed to be 100% identical every single time.

### Secure
All disk access is read-only to ensure the volume is never modified. Writes are handled by a tmpfs overlay with a 
configurable amount of swap space, living in a separate ephemeral volume wiped at every boot.

## Getting Started

### Prerequisites

#### On AWS
To get started, all new you need is an IAM user with at least the following IAM policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "vmify",
            "Effect": "Allow",
            "Action": [
                "ebs:StartSnapshot",
                "ebs:PutSnapshotBlock",
                "ebs:CompleteSnapshot",
                "ec2:DescribeSnapshots",
                "ec2:RegisterImage"
            ],
            "Resource": "*"
        }
    ]
}
```

#### On your machine
Now add the users' credentials to a new `[vmify]` section in `~/.aws/credentials` (the same file used by the AWS CLI):
```ini
[vmify]
aws_access_key_id = AKIAXXXXXXXXXXXXXXXX
aws_secret_access_key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Finally, make sure you have
- a working Docker installation as VMify will leverage that to pull, inspect, squash and
extract your Docker image
- internet access as VMify will need it to upload and register your AMI on AWS

### Usage

```
🚀 VMify
From Docker Image to Cloud in Seconds

Usage: 
  vmify [args] image

AWS credentials are retrieved from the [vmify] profile in ~/.aws/credentials

Examples:
  vmify nginx
  vmify -d -r=us-east-1 -s:dev.hpet.max-user-freq=64 -s:vm.panic_on_oom=0 nginx:latest

Params:
  image       The Docker image to use
  
Args:
  -q          Quiet mode. Only print AWS AMI id when finished
  -t          Timing information suppressed from output
  -d          Debug output turned on during boot
  -b          reBoot instead of terminating upon entrypoint exit
  -r=region   Region in AWS to use (default: eu-central-1)
  -w=number   sWap size (in GiB) to use, 0 to disable swap (default: 1)
  -k=args     Kernel arguments (default: quiet)
  -s:key=val  Sysctl to set with this value
  -h or -?    Show this help message
```

## Supported AWS infrastructure

### Regions

VMify works with the following AWS regions:
- `af-south-1`
- `ap-east-1`
- `ap-northeast-1`
- `ap-northeast-2`
- `ap-northeast-3`
- `ap-south-1`
- `ap-southeast-1`
- `ap-southeast-2`
- `ap-southeast-3`
- `ca-central-1`
- `eu-central-1`
- `eu-north-1`
- `eu-south-1`
- `eu-west-1`
- `eu-west-2`
- `eu-west-3`
- `me-south-1`
- `sa-east-1`
- `us-east-1`
- `us-east-2`
- `us-west-1`
- `us-west-2`

### Instance types

AMIs created by VMify are compatible with the following Intel and AMD instances types:
- `t3`
- `t3a`
- `m6i`
- `m5`
- `m5a`
- `m5n`
- `m5zn`
- `c6i`
- `c6a`
- `c5`
- `c5a`
- `c5n`
- `r5`
- `r5b`
- `r5a`
- `r5n`