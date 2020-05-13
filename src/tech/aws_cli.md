# AWS CLI crib notes

AWS CLI isn't very exciting, but it's helpful.

## Starting and Stopping Instances with Wait

You can start and stop instances.  Take note of the `wait instance-stopped` command, which makes this little trick good.

```sh
aws ec2 stop-instances --instance-ids $INSTANCE_ID 
aws ec2 wait instance-stopped --instance-ids $INSTANCE_ID
aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID --instance-type $INSTANCE_TYPE 
aws ec2 start-instances --instance-ids $INSTANCE_ID
```

## Attach an Elastic IP to an AWS Instance using CLI

```sh
aws ec2 allocate-address --domain vpc  # returns an eipalloc
aws ec2 associate-address --instance-id i-aaaaaaaaaaaa --allocation-id eipalloc-000000000000
```

## Destroy Your AMIs and Their Snapshots!

We frequently create AMIs with packer. This script can help clean
up the images and their associated snapshots.

```sh
aws ec2 describe-images --owners self|grep ami
aws ec2 deregister-image --image-id ami-0000aaaa
aws ec2 describe-snapshots --owner self | grep snap-
aws ec2 delete-snapshot --snapshot-id snap-aaaa0000
```
