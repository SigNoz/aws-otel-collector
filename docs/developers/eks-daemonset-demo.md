### Using AWS-OTel-Collector on Amazon EKS in DaemonSet

This example will introduce how to use AWS-OTel-Collector to send application traces and metrics on AWS EKS in DaemonSet mode. This instruction provided the data emitter image that will generate OTLP format of metrics and traces data to AWS CloudWatch and X-Ray consoles.  Please follow the steps below to try AWS OTel Collector.

### Prerequisite
You can setup EKS cluster for this demo on your AWS account by following [EKS Getting Started Guide](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html). 

### Create EKS-AWSOTel IAM Policy 
1. Open the IAM console at https://console.aws.amazon.com/iam/.
2. In the navigation pane, choose **Policies**.
3. Choose **Create policy, JSON**.
4. Enter the following policy:
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"logs:PutLogEvents",
				"logs:CreateLogGroup",
				"logs:CreateLogStream",
				"logs:DescribeLogStreams",
				"logs:DescribeLogGroups",
				"xray:PutTraceSegments",
				"xray:PutTelemetryRecords",
				"xray:GetSamplingRules",
				"xray:GetSamplingTargets",
				"xray:GetSamplingStatisticSummaries",
				"ssm:GetParameters"
			],
			"Resource": "*"
		}
	]
}
```
5. Choose Review policy.
6. On the Review policy page, enter `EKS-AWSOTel` for the Name and choose Create policy.

#### Attach EKS-AWSOTel IAM Role to worker nodes
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2. Select one of the worker node instances and choose the IAM role in the description.
3. On the IAM role page, choose Attach policies.
4. In the list of policies, select the check box next to `EKS-AWSOTel`. If necessary, use the search box to find this policy.
5. Choose Attach policies.

#### Deploy AWSOTelCollector on Amazon EKS as Service

1. Create a Kubernetes namespace.
```bash
kubectl create namespace aws-otel-eks-daemonset
```
2. An example config template can be found [here](../../examples/eks/eks-daemonset.yaml). Replace `region` value with the region where you expect the data to be published (default, `us-west-2`).
3. Deploy the application.
```bash
kubectl apply -f examples/eks/eks-daemonset.yaml
```
4. View the resources in the `aws-otel-eks-daemonset` namespace.
```bash
kubectl get all -n aws-otel-eks-daemonset
```
5. View the details of the deployed deployment.
```bash
kubectl -n aws-otel-eks describe deployment aws-otel-eks-daemonset
```

The example template provided runs the AWS-OTel-Collector in DaemonSet to send application metrics and traces on Amazon EKS. 
We run two applications: the customer’s application (`aws-otel-sample-app`) and the AWSOTelCollector `aws-otel-collector`. 
Running the AWSOTelCollector in DaemonSet with the same EKS cluster to collect the metric/trace data for the customer’s application from the sample app.

**View Your Metrics**  
You should now be able to view your metrics in your [CloudWatch console](https://console.aws.amazon.com/cloudwatch/). In the navigation bar, click on **Metrics**. The collected AWSOTelCollector metrics can be found in the **AWSObservability/CloudWatchOTService** namespace. Ensure that your region is set to the region set for your cluster.