# AWS CLI crib notes

AWS CLI isn't very exciting, but it's helpful.

You can start and stop instances.  Take note of the `wait instance-stopped` command, which makes this little trick good.

```sh
aws ec2 stop-instances --instance-ids $INSTANCE_ID 
aws ec2 wait instance-stopped --instance-ids $INSTANCE_ID
aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID --instance-type $INSTANCE_TYPE 
aws ec2 start-instances --instance-ids $INSTANCE_ID
```
