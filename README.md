# POI detection with Amazon Nova Pro

Detecting features in images, like points of interest in aerial footage is a
frequent task in many fields. However, traditional approaches - whether manual
review or custom trained computer vision models - are costly, slow to adapt and
difficult to scale across diverse use cases.

This Jupyter notebook introduces a zero-training, no-dataset-required visual
detection system using Amazon Nova Pro, a multimodal foundation model accessed
via Amazon Bedrock. Using only a Jupyter notebook, you can detect points of
interest with natural language prompts. You do not need computer vision
expertise or labeled data sets.

By the end of this notebook, you'll be able to:

* Download and prepare and analyze aerial footage
* Detect points of interest using Amazon Nova Pro
* Draw bounding boxes around your point of interest
* Calculate coordinates form pixel positions

## Prerequisites

If you want to follow along or reproduce this example, you will need an [AWS Account](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html), the [AWS CLI](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html) setup with valid [IAM-Credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html). If you haven’t installed and set up your AWS CLI please follow the official [AWS Documentation.](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)

### IAM Permissions for Amazon Bedrock

In order to use Amazon Bedrock, you need an IAM Role or an IAM User with the appropriate permissions. The minimum required permissions are `bedrock-runtime:InvokeModel` and potentially `bedrock-runtime:InvokeModelWithResponseStream` to call the LLM. Here is an example of a minimal IAM policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock-runtime:InvokeModel"
            ],
            "Resource": "arn:aws:bedrock:[YOUR_REGION]::foundation-model/amazon.nova-pro-v1:0"
        }
    ]
}
```

### Request Model Access

Requesting model access is required for each model in each region. If you haven’t setup model access please navigate in your AWS Console to the Amazon Bedrock service and choose [“Model Access” in the left menu](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access-modify.html) and request access for the model you want to use.

### Deployment

You can run the Jupyter Notebook locally or in an Amazon SageMaker Notebook Instance. To deploy a Notebook Instance use the CloudFormation stack with the AWS CLI.

```bash
aws cloudformation create-stack \
  --stack-name GenerativePOIDetection \
  --template-body file://Notebook-Cloudformation.yaml \
  --capabilities CAPABILITY_NAMED_IAM && \
aws cloudformation wait stack-create-complete --stack-name GenerativePOIDetection && \
aws cloudformation describe-stacks --stack-name GenerativePOIDetection --query 'Stacks[0].Outputs[?OutputKey==`NotebookURL`].OutputValue' --output text

```

Or you can click this button to automatically deploy the template to your AWS Account [![Deploy to AWS](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=GenerativePOIDetection&templateURL=https://raw.githubusercontent.com/aws-samples/sample-generative-poi-detection/main/Notebook-Cloudformation.yaml)

Once the deployment is finished click on the Link in your Console or Navigate to your Amazon CloudFormation Stack Output Tab and look for the `NotebookURL` output. Click on the link to open the Jupyter Notebook Instance.

### Cleanup

Once you are finished you can delete the CloudFormation stack to remove the ressources you deployed.

```bash
aws cloudformation delete-stack --stack-name GenerativePOIDetection
```

## Pipeline Architecture in this notebook

This inspection pipeline operates entirely within a local Jupyter notebook and AWS Generative AI infrastructure:

    Retrieve aerial footage: Get an aerial image from the area you want. This example uses SWISSIMAGE from Swisstopo.
    Image Preprocessing: Images are resized, split into chunks, converted to Base64, and prepared for inference.
    Generative AI Inference: Amazon Bedrock invokes Nova Pro to analyze the image and return structured POI data.
    Visualization: Bounding boxes are drawn using matplotlib. Pixel position is converted to coordinates

User → Jupyter Notebook → Amazon Bedrock (Nova Pro) → JSON POI Output → Matplotlib Overlay

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.