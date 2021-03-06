![MIE logo](doc/images/MIE_logo.png)

Media Insights Engine (MIE) is a framework to accelerate the development of serverless applications that process video, images, audio, and text with artificial intelligence services and multimedia services on AWS. MIE is most often used to: 

1. Create media analysis workflows using [Amazon Rekognition](https://aws.amazon.com/rekognition/), [Amazon Transcribe](https://aws.amazon.com/transcribe/), [Amazon Translate](https://aws.amazon.com/translate/), [Amazon Cognito](https://aws.amazon.com/cognito/), [Amazon Polly](https://aws.amazon.com/polly/), and [AWS Elemental MediaConvert](https://aws.amazon.com/mediaconvert/).
2. Build analytical applications on top of data extracted by workflows and saved in the [Amazon Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/)

MIE includes a demo GUI for video content analysis and search. The [Implementation Guide](https://github.com/awslabs/aws-media-insights-engine/blob/master/IMPLEMENTATION_GUIDE.md) explains how to build other applications with MIE. 

![](doc/images/MIEDemo.gif)

The Media Insights sample application lets you upload videos, images, audio and text files for content analysis and add the results to a collection that can be searched to find media that has attributes you are looking for.  It runs an MIE workflow that extracts insights using many of the ML content analysis services available on AWS and stores them in a search engine for easy exploration.  A web based GUI is used to search and visualize the resulting data along-side the input media.  The analysis and transformations included in MIE workflow for this application include:

* Proxy encode of videos and separation of video and audio tracks using **AWS Elemental MediaConvert**. 
* Object, scene, and activity detection in images and video using **Amazon Rekognition**. 
* Celebrity detection in images and video using **Amazon Rekognition**
* Face search from a collection of known faces in images and video using **Amazon Rekognition**
* Facial analysis to detect facial features and faces in images and videos to determine things like happiness, age range, eyes open, glasses, facial hair, etc. In video, you can also measure how these things change over time, such as constructing a timeline of the emotions expressed by an actor.  From **Amazon Rekognition**.
* Unsafe content detection using **Amazon Rekognition**. Identify potentially unsafe or inappropriate content across both image and video assets. 
* Convert speech to text from audio and video assets using **Amazon Transcribe**.
* Convert text from one language to another using **Amazon Translate**.
* Identify entities in text using **Amazon Comprehend**. 
* Identify key phrases in text using **Amazon Comprehend**
* Locate technical cues such as black frames, end credits, and color bars in your videos using Amazon Rekognition.
* Identify start, end, and duration of each unique shot in your videos using Amazon Rekognition. 

Data are stored in Amazon Elasticsearch Service and can be retrieved using _Lucene_ queries in the Collection view search page.

# Installation
You can deploy MIE and the demo GUI in your AWS account with the following instructions:

#### *Step 1. Launch the Stack*
  Region| Launch
  ------|-----
  US East (N. Virginia) | [![Launch in us-east-1](doc/images/launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=mie&templateURL=https://rodeolabz-us-east-1.s3.amazonaws.com/media-insights-solution/v0.1.8/cf/media-insights-stack.template)
  US West (Oregon) | [![Launch in us-west-2](doc/images/launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=mie&templateURL=https://rodeolabz-us-west-2.s3.amazonaws.com/media-insights-solution/v0.1.8/cf/media-insights-stack.template)

1. Sign in to the AWS Management Console in either the US-East-1 or US-West-2 regions.
2. Select a template to launch from the table above based on the region you are signed into. This will take you to the 
Cloudformation deployment menu. 
3. On the Create stack page, verify that the correct template URL shows in the Amazon
S3 URL text box and choose Next.
4. On the Specify stack details page, assign a name to your MIE stack.
5. Under Parameters, review the parameters for the template and modify them as
necessary. The default settings for the template will deploy MIE and the demo GUI. You must set the parameters for `Stack name` and `AdminEmail`.
6. Choose Next.
7. On the Configure stack options page, choose Next.
8. On the Review page, review and confirm the settings. Check the box acknowledging that
the template will create AWS Identity and Access Management (IAM) resources.
9. Choose Create stack to deploy the stack

You can view the status of the stack in the AWS CloudFormation Console in the Status
column. You should see a status of CREATE_COMPLETE in approximately 15-30 minutes. 

For more information about stack deployment, see the section on [installation parameters](#installation-parameters).

#### *Step 2. Access the Web Application*

To access the web application, you will need the email address you provided in the AWS
CloudFormation template, a temporary password, which is emailed to your specified
address, and the URL of your deployed MIE sample application.

In the same email that includes your temp password, there is a link to the MIE stack overview.

1. Click that link to access the stack overview page.
2. Navigate to the `Outputs` tab of the MIE stack.
3. Copy the value of the `MediaInsightsWebAppUrl` output. This is the URL you will use to access the sample application.
4. In a new browser tab, paste the `MediaInsightsWebAppUrl` value into the navigation bar and access the app.
5. Sign in with the email address you provided and the temp password from the automated email. You will be prompted to set a new password.
6. Upload some content from the `Upload` link in the navigation header and explore the application.

## Outputs

After the stack successfully deploys, you can find important interface resources in the **Outputs** tab of the MIE CloudFormation stack.

**MediaInsightsWebAppUrl** is the Url to access sample Media Insights web application

**DataplaneApiEndpoint** is the endpoint for accessing dataplane APIs to create, update, delete and retrieve media assets

**DataplaneBucket** is the S3 bucket used to store derived media (_derived assets_) and raw analysis metadata created by MIE workflows.

**ElasticsearchEndpoint** is the endpoint of the Elasticsearch cluster used to store analysis metadata for search

**MediaInsightsEnginePython37Layer** is a lambda layer required to build new operator lambdas

**WorkflowApiEndpoint** is the endpoint for accessing the Workflow APIs to create, update, delete and execute MIE workflows.

**WorkflowCustomResourceArn** is the custom resource that can be used to create MIE workflows in CloudFormation scripts


# Cost

MIE itself does not have a significant cost footprint. The MIE control plane and data plane generally cost less than $1 per month. However, when people talk about the cost of MIE they're generally talking about the cost of running some specific application that was built on top of MIE. Because those costs can vary widely you will need to get pricing information from the documentation for those applications. As a point of reference, see the README for the Content Analysis application that is included under the webapp directory.

# Limits

The latest MIE release has been verified to support videos up to 2 hours in duration. 

# Architecture Overview

Media Insights Engine is a _serverless_ architecture on AWS. The following diagram is an overview of the major components of MIE and how they interact when an MIE workflow is executed.  

![](doc/images/MIE-execute-workflow-architecture.png)


## Workflow API
Triggers the execution of a workflow. Also triggers create, update and delete workflows and operators.  Monitors the status of workflows.

## Control plane
Executes the AWS Step Functions state machine for the workflow against the provided input.  Workflow state machines are generated from MIE operators.  As operators within the state machine are executed, the interact with the MIE data plane to store and retrieve derived asset and metadata generated from the workflow.  

## Operators
Generated state machines that perform media analysis or transformation operation.

## Workflows
Generated state machines that execute a number of operators in sequence.

## Data plane
Stores media assets and their associated metadata that are generated by workflows. 

## Data plane API

Trigger create, update, delete and retrieval of media assets and their associated metadata.

## Data plane pipeline

Stores metadata for an asset that can be retrieved as a single block or pages of data using the objects AssetId and Metadata type.  Writing data to the pipeline triggers a copy of the data to be stored in a **Kinesis Stream**.

### **Data plane pipeline consumer**

A lambda function that consumes data from the data plane pipeline and stores it (or acts on it) in another downstream data store.  Data can be stored in different kind of data stores to fit the data management and query needs of the application.  There can be 0 or more pipeline consumers in a MIE application. 

# Installation Parameters

You can deploy MIE and the demo GUI in your AWS account with the [one-click deploy buttons](#installation) shown above. 

## Required parameters

**Stack Name**: Name of stack. Defaults to `mie`.

**System Configuration**
* **MaxConcurrentWorkflows**: Maximum number of workflows to run concurrently. When the maximum is reached, additional workflows are added to a wait queue. Defaults to `10`.

**Operators** 
* **Enable Operator Library Deployment**: If set to true, deploys the operator library. Defaults to `true`.

**Workflows**
* **DeployTestWorkflow**: If set to true, deploys test workflow which contains operator, stage and workflow stubs for integration testing. Defaults to `false`.
* **DeployInstantTranslateWorkflow**: If set to true, deploys Instant Translate Workflow which takes a video as input and transcribes, translates and creates an audio file in the new language. Defaults to `false`.
* **DeployRekognitionWorkflow**: If set to true, deploys Rekognition Workflows which process videos and images through Rekognition, Transcribe, Translate, etc. Defaults to `true`.
* **DeployComprehendWorkflow**: If set to true, deploys a Comprehend Workflow which takes text as input and identifies key entities and phrases. Defaults to `false`.
* **DeployKitchenSinkWorkflow**: If set to true, deploys the Kitchen Sink Workflow which contains all MIE operators. Defaults to `true`.

**Sample Applications**
* **DeployDemoSite**: If set to true, deploys a front end application to explore extracted metadata. Defaults to `true`.

**Other parameters**
* **DeployAnalyticsPipeline**: If set to true, deploys a metadata streaming pipeline that can be consumed by downstream analytics plaforms. Defaults to `true`.

### Example use cases for Media Insights Engine
 
MIE is a reusable architecture that can support many different applications.  Examples:
 
* **Content analysis analysis and search** - Detect objects, people, celebrities and sensitive content, transcribe audio and detect entities, relationships and sentiment.  Explore and analyze media using full featured search and advanced data visualization.  This use case is implemented in the included sample application.
* **Automatic Transcribe and Translate** - Generate captions for Video On Demand content using speech recognition.  
* **Content Moderation** - Detect and edit moderated content from videos.

# Developers

Join our Gitter chat at [https://gitter.im/awslabs/aws-media-insights-engine](https://gitter.im/awslabs/aws-media-insights-engine). This public chat forum was created to foster communication between MIE developers worldwide.

[![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/awslabs/aws-media-insights-engine)

MIE is built to be extended in the following ways:

* Run existing workflows with custom  configurations.
* Create new operators for new types of media analysis or transformation
* Create new workflows using the existing or new operators.
* Stream data to new data storage services, such as Elasticsearch or Amazon Redshift.

See the [Implementation Guide](https://github.com/awslabs/aws-media-insights-engine/blob/master/IMPLEMENTATION_GUIDE.md) for the MIE API reference and builder's guide.

# Known Issues

Visit the Issue page in this repository for known issues and feature requests.

# Contributing

See the [CONTRIBUTING](CONTRIBUTING.md) file for how to contribute.

# Logo

The [MIE logo](doc/images/MIE_logo.png) features a clapperboard representing *multimedia*, centered inside a crosshair representing *under scrutiny*. 

# License

See the [LICENSE](LICENSE.txt) file for our project's licensing.

Copyright 2020 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
