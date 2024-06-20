# AWS-Cost-Optimization-Project
Cost Optimization of AWS Infrastructure by automating the deletion of stale EBS Snapshots


In the following cases, the Snapshots need to be deleted :

Case 1 : If the Snapshot is created and it is not attached to any Volume.

Case 2 : If the Snapshot is created and it is attached to a Volume, but that Volume is not attached to an EC2 Instance.

### PROJECT OVERVIEW :

The Lambda Function would watch for unused / stale EBS Snapshots and delete them. 

The Lambda Function is created with Python and by using the boto3 module, it is possible to interact with the AWS Service APIs. 

The Lambda Function can be triggered through CloudWatch Events. 

The final outcome of the project will help the Cloud Engineers reduce the unwanted expenses in the cloud.

As a result, Cost Optimization can be achieved through this project.


### PROJECT WORKFLOW :

### Creating an EC2 Instance, EBS Volume and Snapshots :

![WhatsApp Image 2024-06-20 at 09 37 08_94fa6a62](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/a2a0cfe9-a656-4b02-ba93-50065a390c9f)

![WhatsApp Image 2024-06-20 at 09 37 09_e1e386a1](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/ce08f6a2-2ace-4165-9501-26c6aabc9a3a)

![WhatsApp Image 2024-06-20 at 09 37 09_b81183c4](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/ff3d4d6f-4a4c-4a98-a32d-13ceaa48f2ff)

### EC2 Dashboard at Initial Stage :

![WhatsApp Image 2024-06-20 at 09 37 09_e11fcb15](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/cdece7e8-28bb-42ba-a9f8-f52de13cb5f2)

### Creation of Lambda Function :

```

import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # Get all EBS snapshots
    response = ec2.describe_snapshots(OwnerIds=['self'])

    # Get all active EC2 instance IDs
    instances_response = ec2.describe_instances(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}])
    active_instance_ids = set()

    for reservation in instances_response['Reservations']:
        for instance in reservation['Instances']:
            active_instance_ids.add(instance['InstanceId'])

    # Iterate through each snapshot and delete if it's not attached to any volume or the volume is not attached to a running instance
    for snapshot in response['Snapshots']:
        snapshot_id = snapshot['SnapshotId']
        volume_id = snapshot.get('VolumeId')

        if not volume_id:
            # Delete the snapshot if it's not attached to any volume
            ec2.delete_snapshot(SnapshotId=snapshot_id)
            print(f"Deleted EBS snapshot {snapshot_id} as it was not attached to any volume.")
        else:
            # Check if the volume still exists
            try:
                volume_response = ec2.describe_volumes(VolumeIds=[volume_id])
                if not volume_response['Volumes'][0]['Attachments']:
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as it was taken from a volume not attached to any running instance.")
            except ec2.exceptions.ClientError as e:
                if e.response['Error']['Code'] == 'InvalidVolume.NotFound':
                    # The volume associated with the snapshot is not found (it might have been deleted)
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as its associated volume was not found.")

```

### Execution Result of Lambda Function at Initial Stage : 

![WhatsApp Image 2024-06-20 at 09 37 10_38683eb9](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/dd78c128-ba35-4f98-a176-2f1b850be2a2)


Upon testing the Lambda Function, it failed because the default execution time of Lambda Function if only 3 Seconds. 

Another reason for the failure is also lack of appropriate Permissions to describe the Snapshots.

Because the task we are performing is Describing all the Snapshots and Filtering out the Stale Snapshots.

So, the above issue can be rectified by increasing the default Execution Time of the Lambda Function to 10 seconds.

### Granting the Appropriate Permissions :

For the successful execution of the Lambda Function, the following permissions need to be given :-

1) Describe Snapshots

2) Delete Snapshots


### Successful execution of the Lambda Function :

![WhatsApp Image 2024-06-20 at 09 37 10_2356ad88](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/773c1a28-d6bf-43e3-92be-2f675bfcdea5)

Though the Lambda Function got executed successfully, the Snapshots will not be deleted. 

![WhatsApp Image 2024-06-20 at 09 37 11_8166fc35](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/1919619b-6f3c-4023-943d-b27fbb79bd19)

![WhatsApp Image 2024-06-20 at 09 37 11_52aa2d8d](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/3853868f-0c20-4b38-971d-e97a65120308)

### Deletion of the Snapshot :

Step 1 : Terminate the EC2 Instance. As a result, the EBS Volume attached to the EC2 Instance also got deleted. But the Snapshot still Remains (NOT DELETED).

![WhatsApp Image 2024-06-20 at 09 37 12_9b0f9521](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/a0e3dd0f-5c5c-42bb-b229-3fa527a595e2)

![WhatsApp Image 2024-06-20 at 09 37 12_a471de62](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/02c96e94-fc7a-4c4e-8954-5c71665463bf)

![WhatsApp Image 2024-06-20 at 09 37 12_76e12339](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/ab08c5d1-a49e-4b54-89bf-ca1821c5c31c)

Upon executing the Lambda Function, the Snapshot got deleted. The execution of the Lambda Function can be done manually (or) can be triggered through Amazon CloudWatch Events.

![WhatsApp Image 2024-06-20 at 09 37 13_e49f0fbf](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/703bd158-3db4-4e5d-8d01-478cb6b9a8f3)

![WhatsApp Image 2024-06-20 at 09 37 13_841a6b1b](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/9e7baa31-d943-4d13-b1aa-a4878629e772)


The Snapshot is considered as a stale Snapshot, because its associated Volume was deleted.

### Final EC2 Dashboard (At the end of the Project) :

![WhatsApp Image 2024-06-20 at 09 37 14_b8715105](https://github.com/vighas-ks-16/AWS-Cost-Optimization-Project/assets/107311113/7ecb2e77-6b8f-4825-942a-ac80c36eba04)



### CONCLUSION :

Thus Cloud Cost Optimization can be done by an AWS and DevOps Engineer in an Organization.

