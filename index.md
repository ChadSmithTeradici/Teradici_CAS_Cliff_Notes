---
title: Teradici CAS Cliff Notes
description: A guide deployed to quickly set-up CAS on a workstation without having to consult official documentation
author: chad-m-smith
tags: CAS, CAS-Plus, Windows
date_published: 2022-02-15
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

While Teradici's [official documentation](https://docs.teradici.com/find/product/cloud-access-software) is the recommended for deploying CAS, this “cliff noted” guide is a condensed version, highlighting recommendations, troubleshooting and observations 'in-line' as a part of this deployment process. This removes the need to open several  guides and ‘cuts the fat’ and illustrates the most common deployment scenarios seen out in the field over the past two years at Teradici.

**A brief description of deployment:**
1. Review of system requirement to ensure compatibility
1. Administrator of workstation logs into Teradici portal to download CAS agent.
1. Install CAS agent on workstation
1. Configure CAS agent
1. End-user logs into Teradici portal to download CAS client on their remote system
.
# Decide on the version of CAS agent that is needed #
**A assumption on what type of CAS agent and purchase has been made **
There are two CAS agent SKUs availble depending on the requirements of the end-user. Each of these SKUs have a difference price point that is assoicated to the purchase/ registration key at time of sale. It is important that right agent is selected, there have been situations where end-users paid for Graphics Agents but downloaded Standard Agent. Our licensing server would permit a user to register the workstation, but it would prevent the opposite (paying for Standard but installing Graphics) from happening.

## Create an IAM Role ##
Create an [IAM role](https://console.aws.amazon.com/iamv2/home?#/roles) to provide the proper permissions for the end-user to start and connect to defined instances.

1. Go to IAM -> Role -> Create Role. 
    
1. Select the **Create role** button, select the **Lamdba**

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CreateRole.jpg)
    
Select the **Next Permissions** button.

1. In Filter policies type in 'EC2Full' and Select the **AmazonEC2FullAccess**

    **Note:** Selecting Full access isn't recommended for most production environment, consult with your security team on narrowing down permission that this           service will run.
    
    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AttachPermission.jpg)    
    
    Select **Next:Tags**
  
1. After *Option* Tag section, The role review page has you name the role. In this example it was named *Lamdba_EC2_long_running_instance*. 

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CreateRoleReview.jpg)

## Create an Lamdba function ##
Next we will create a Lambda function (λ). The function does two things:
- Get the EC2 instance ID set focus on only G4dn instance types
- Create an alarm and attach it to the EC2 instance

1. Select a AWS region that has EC2 Instance type available, the Lamdba function (λ) is designed to run in the same region. If you have other instances in other regions / loca zones, you should duplicate this setup as well. Enter the [Lamda dashboard](console.aws.amazon.com/lambda/) and select the **Create function** button

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CreateFunction.jpg)

1. Within the create function wizard, select the **Author from scratch** option.

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AuthorFromScatch.jpg)

1. Fill in the basic Lamdba funcation (λ) information:
- Our example we named the Lamdba funcation **Stop_idle_EC2-G4dn_instance**
- The Runtime field enter **Python 3.6**
- Select Architecture type as: **x86_64**
- Under Permission > Change default execution role > **Use existing role** > Choose the previously created IAM role above
- Once the required fields are filled out, Select the **Create funcation** button to continue.

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/createFuncationBasic.jpg)

4. Within the funcation overview dashboard, we will start by adding the Lamdba event **trigger** 
- In the 'Select a trigger' box, select the **EventBridge (CloudWatch Events)** type
- Under **Rule**, pick the **Create a new rule** option
- Enter a **Rule name** to identify your rule, example : **Stop_idle_EC2-G4dn_Instances**
- Under **Rule type**, select **Event pattern**
- Use 'drop down box' to select **EC2**
- Use second 'drop down box'to select **EC2 instance state-change notification**
- 'Click' **Detail** to expand options
- Ensure that the **State** box is 'checked'
- Within the **State** field, auto-complete the word **running**
- Once complete select the **Add** on botton of page

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AddTrigger1.jpg)

When successfully completed, a notification will appear says that all subsystem between services have been established.

   ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/SuccessfulTrigger.jpg)

5. Finally, drop in the Python code into the **Code Source** section of the Lamdba Dashboard. Ensure you are in the **Code** section of console.

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CodeToolBar.jpg)

- Remove any pre-populated python code
- **Copy** the snippet of python code below
- Ensure that **line 9** *AlarmActions* has the correct AWS region setup (Example: this code is focused on us-west-2),change if needed.
``` AlarmActions       = ['arn:aws:automate:us-west-2:ec2:stop'],```

```
import boto3


def put_cpu_alarm(instance_id):
    cloudWatch   = boto3.client('cloudwatch')
    cloudWatch.put_metric_alarm(
        AlarmName          = f'CPU_ALARM_{instance_id}',
        AlarmDescription   = 'Alarm when server CPU does not exceed 5%',
        AlarmActions       = ['arn:aws:automate:us-west-2:ec2:stop'],
        MetricName         = 'CPUUtilization',
        Namespace          = 'AWS/EC2' ,
        Statistic          = 'Average',
        Dimensions         = [{'Name': 'InstanceId', 'Value': instance_id}],
        Period             = 600,
        EvaluationPeriods  = 3,
        Threshold          = 5,
        ComparisonOperator = 'LessThanOrEqualToThreshold',
        TreatMissingData   = 'notBreaching'
    )


def lambda_handler(event, context):
    instance_id = event['detail']['instance-id']
    ec2 = boto3.resource('ec2')
    instance = ec2.Instance(instance_id)
    if instance.instance_type.startswith('g'):
        put_cpu_alarm(instance_id)
```
6. In the **Code Source** text editor, select the **File** > **Save** option

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/SavedPythonCode.jpg)

7. Finally, you want to deploy the saved code to run as your Lamdba function (λ). To do this select the **Deploy** button ontop of the text editor tool.

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/DeployPythonCode.jpg)
   
**Note** there is notifcation status that will change from **Changes not deployed** to **Changes deployed**

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/PythonSuccDeploy.jpg)

# Monitoring EC2 G-family idle resource shut down workflow #

The EC2 Dashboard shows summary information aboutCloudWatch alarms per instance.

You get some general information about an alarm being assigned to the instance, by verifying that the Lambda Function (λ) did assigned a alarm to the instance with the **1 in alarm** in the alarm status column.

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AlarmSet.jpg)

The alarm status will change to **1/1 in alarm** and change to red front when the threshold criteria has been met and the instance was shutdown, this is an easy see with the EC2 Dashboard that CloudWatch event triggered a shutdown without leaving the console. 

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AlarmAction.jpg)

A more detailed view of the CloudWatch alarms are in the [CloudWatch Dashboard](console.aws.amazon.com/cloudwatch/), which prodives more details on the state of the instances. There may be other alarms active, so you should be note the instance-id of the instance associated alarm you are want to look at. EC2 alarms are referenced to by its instance-ids. In the alarm dashboard, the red line represents the 7% threshold, while the blue line represents the number of polls take and their associated CPU utilization.  The status bar under the graph represents alarm stat, green represents that event criteria can't been meet. A red bar represents the threshold has been meet and in this CloudWatch event, means the instance was shutdown. 

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CloudWatchAlarm.jpg)

# Disabling EC2 G-family idle resource shut down workflow #

There are two ways to disable alarms and the associated idle power management either at an instance level or across the entire region. The first method is to manually delete the alarm after the EC2 instance is launched. This is done on a per-instance basis and if the instance stays in a powered-on state and the alarm has been deleted the instance will not be monitored and not shutdown when the CPU utilization dips. Be aware, if the instance is manually cycled, that change in power state will be detected by the Lambda function and reapply the CloudWatch alarm.

To manually delete an alarm enter the [CloudWatch Dashboard](console.aws.amazon.com/cloudwatch/), on the left-hand side select **All Alarms**. All alarms are associated to the instance-id of the EC2 instance, so you have correlate the your EC2 instance-id to the Alarm name. When you find the assoicated alarm, 'click' on the ckeck-box next to the Name, then slect the **Actions** menu and scroll to **Delete**.

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/RemoveAlarm_While_running.jpg)

To disable a region wide idle resource management, the best approach is go into the [IAM role]( console.aws.amazon.com/iam/home#/roles/) and apply and explicit deny to the IAM policy the Lambda function used to assign a CloudWatch alarm. The Lamdba function will report an permission error, everytime a EC2 chnages states because its ability to monitor and shutdown instances have been removed. When no longer needed, you can remove the revoke rule, to continue Idle resourse management across the region. 

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/RevokeAccess.jpg)
