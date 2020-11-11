# Automatic multiregion replication for ECR repositories

AWS ECR is a great service offered by AWS to have a fully managed docker registery for storing images. Apart from standard storage capabilities, it also offers unique features like Vulnerability scanning, immutable configuration on image tags, image lifecycle policies etc.

### TODO - Template explaination
This template deploys the following set of resource across 3 regions.

TODO: make screenshot using draw.io which contains
Cloud watch events, code buid steps and many more things

### TODO - Sequence of events

#TODO: Insert a process flow digram using draw.io

### How to test
Step 1: Get a sample image you want to test with
```bash
docker pull python:3-alpine
```
Step 2: Tag images based on the region we are going to push them to:
Observed that the image names indicate the source. This is done to make it easy to track the creation & push to other regions which will be peformed by the replication action.

US-EAST-1
```bash
docker tag python:3-alpine <account-id>.dkr.ecr.us-east-1.amazonaws.com/multiregion-ecr-test-src-us-east-1:1
```
EU-NORTH-1
```bash
docker tag python:3-alpine <account-id>.dkr.ecr.eu-north-1.amazonaws.com/multiregion-ecr-test-src-eu-north-1:1
```
AP-SOUTHEAST-1
```bash
docker tag python:3-alpine <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/multiregion-ecr-test-src-ap-southeast-1:1
```
Step 3: Login to us-east-1

```bash
aws ecr get-login-password --profile <sandbox-profile> --region us-east-1 | docker login --username AWS --password-stdin https://<account-id>.dkr.ecr.us-east-1.amazonaws.com
```

Step 4: Create ecr repository

```bash
aws ecr create-repository --repository-name multiregion-ecr-test-src-us-east-1 --profile=<sandbox-profile>  --region us-east-1
```

Step 5: Push image to newly created ECR repository

```bash
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/multiregion-ecr-test-src-us-east-1:1
```

Step 6: Repeat steps 3 through 5 from eu-north-1

Observations:
After completion of Step 5, the push image action will trigger the AWS CodeBuild Build step which is responsible for replication. You can look at the build step logs in the console.
TODO: insert screenshot

The build step completes execution and creates and pushes new images with the same name and tags in the other two regions.

Same observation can be made based on pushes from either regions.

### Estimating Cost

ECR repo cost (higher level analysis)

Cost 1: What is the cost to executed automated replication?

Assumptions: 
1. There are 3000 pushes total to the docker registry total.
2. We are using general1.small instance type to run the build step

Consider we have 1000 each pushes from 3 regions thats total 3000 pushes
Free per month: Each region get 100 build minutes free per months.

So code of the deploy step which pushes to other 2 regions overall would be calculated as:
((Time to deploy to 2 other regions (which is approximately 2 minutes ) x Number of pushes)-free minutes per region) x Number of regions

TODO: Please calculate time based on compute instance for that python image

Answer Cost 1: Cost for replication step: (((1000*2)-100)*0.01)*3 = $28.5

Cost 2: What are ECR costs?<br>

This include 2 aspects data transfer and data storage.
Transfer = Data in ($0.00) + Data Out (Total data transfer out in GBs - 1 GB free)*$0.09
Storage  = $ 0.10 per GB per Month

Assumption: 100 GB docker image repo size total

Cost 2: Total ECR cost = Data Transfer cost -> 99*0.09 + Storage -> 100*0.10 =$9.91

Total cost = $57+$9.91 = 66.91

Note: Since AWS is so competitive with cloud costs, please refer the following link to get accurate costs:<br>
Code Build Pricing: https://aws.amazon.com/codebuild/pricing/<br>
ECR Pricing: https://aws.amazon.com/ecr/pricing

Note: The build time of the CodeBuild step depends on the compute instance used and the size of the image. Inorder to get a good maximum estimate, assume or figure out the maximum size of a docker image you would want to host in ECR. Then run a sample code build step for that image to get datapoint per push which can be extrapolated.

### References

Here are some links which will help in understanding the various concepts used in this project
1. Deploying infrastructure using AWS CodePipeline:<br>
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/continuous-delivery-codepipeline-basic-walkthrough.html

2. This approach that I have taken is heavily influenced from this contribution (I can mostly call my work as a mere extension):<br>
https://github.com/aws-samples/amazon-ecr-cross-region-replication/blob/master/README.md


### License Summary
This sample code is made available under the MIT-0 license. See the LICENSE file.

### Keep contributing to Open Source
लोकाः समस्ताः सुखिनोभवंतु
