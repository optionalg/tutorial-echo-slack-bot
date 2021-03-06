# Tutorial: Running a Slack bot on AWS using coldbrew-cli

This is a sample project to demonstrate how to run a [Slack](https://slack.com/) bot on AWS using [coldbrew-cli](https://github.com/coldbrewcloud/coldbrew-cli).

## Getting Started

### Install Docker

[coldbrew-cli](https://github.com/coldbrewcloud/coldbrew-cli) deploys your applicaiton in [Docker containers](https://www.docker.com/what-docker). So the first step is to [install Docker](https://docs.docker.com/engine/installation/) in your system if you don't have it yet.

### Install coldbrew-cli

Download the package for your system at [here](https://github.com/coldbrewcloud/coldbrew-cli/wiki/Downloads). It makes things easier if you copy the downloaded binary `coldbrew` (or `coldbrew.exe` on Windows) in your `$PATH`, so you can run `coldbrew` from anywhere. _But it's also okay to keep `coldbrew` executable in your application directory if you want._

### AWS Account

As you will be deploying your application on AWS, you will need AWS account for sure. [Sign up](https://aws.amazon.com/) if you haven't yet, and, get your [AWS access keys](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html). You can pass AWS access keys to **coldbrew-cli** either in [environment variables](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Environment-Variables) or using [CLI flags](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Global-Flags), but, we will assume that you set the follow environment variables through out the turorial:

- `$AWS_ACCESS_KEY_ID`: AWS Access Key ID
- `$AWS_SECRET_ACCESS_KEY`: AWS Secret Access Key
- `$AWS_REGION`: AWS region name
- `$AWS_VPC`: AWS VPC ID _(this is completely optional. If you don't specify or don't know your VPC ID, **coldbrew-cli** will automatically use the [default VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/default-vpc.html) of your AWS account.)_

### Slack Bot Account

You will need a Slack bot account for this tutorial. You can [create a Slack bot account](https://my.slack.com/services/new/bot) if you don't have one. Just save he API token of the bot account because that will be used later in this tutorial.

### Clone This Repo

This tutorial project contains the bare minimum _(but fully functional)_ sample resources so you can get started right away.

- A sample Slack bot (written in Go): the bot simply echos what the calling user said
- A sample [Dockerfile](https://github.com/coldbrewcloud/tutorial-echo-slack-bot/blob/master/Dockerfile)
- A sample **coldbrew-cli** app configuration file, [coldbrew.conf](https://github.com/coldbrewcloud/tutorial-echo-slack-bot/blob/master/coldbrew.conf)

Clone this repo:

```bash
git clone https://github.com/coldbrewcloud/tutorial-echo-slack-bot.git
cd tutorial-echo-slack-bot
```

## Creating Your First Cluster

**coldbrew-cli** has _very_ simple [concepts](https://github.com/coldbrewcloud/coldbrew-cli/wiki/Concepts): clusters and applications (apps). An app is the minimum deployment unit and _typically_ it correponds to a project (like this tutorial project). And a cluster is simply a collection of apps who will share some AWS resources. _Most importantly they share the Docker hosts (which is ECS Container Instances in the AWS context)._

Let's create your first cluster `tutorial` using [cluster-create](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-cluster-create) command:

```bash
coldbrew cluster-create tutorial --disable-keypair
```

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-cluster-create.gif?v=2" width="800">

_*In this tutorial, we used `--disable-keypair` flag to skip assigning EC2 key pairs to the container instances. If you will need a direct access to the instances (e.g. via SSH), you can use `--key` flag to specify your key pair name._


If you want to check the current running status of your first cluster, you can use [cluster-status](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-cluster-status) command:

```bash
coldbrew cluster-status tutorial
```

It can take several minutes until the initial ECS Container Instances (EC2 Instances) become fully available, but, you can continue on to start deploying your applicaiton.

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-cluster-status.gif?v=1" width="800">

## Deploying Your Application

Now it's time to deploy your app for the first time. To deploy the applicaiton, you need a [configuration file](https://github.com/coldbrewcloud/coldbrew-cli/wiki/Configuration-File) to define your app's deployment settings. You can easily create it on your own, or, you can use [init](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-init) command to generate a proper default configuration for your app. In this tutorial, we provide a sample [coldbrew.conf](https://github.com/coldbrewcloud/tutorial-echo-slack-bot/blob/master/coldbrew.conf) file.

The sample Bot application takes the Slack API token via an environment variable `$SLACK_API_TOKEN`. So you can set and pass the variable using `.env` attributes in [coldbrew.conf](https://github.com/coldbrewcloud/tutorial-echo-slack-bot/blob/master/coldbrew.conf) file. Just replace

```yaml
env:
  SLACK_API_TOKEN: your_slack_api_token
```

with your actual API token.

Now you can deploy your bot using [deploy](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-deploy) command:

```bash
coldbrew deploy
```

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-deploy.gif?v=2" width="800">

You just ran a single command line, but, what's realy happening here is:
- **coldbrew-cli** builds local Docker image using [Dockerfile](https://github.com/coldbrewcloud/tutorial-echo-slack-bot/blob/master/Dockerfile). _(You can skip this if you provide the local image name using `--docker-image` flag directly.)_
- It pushes the Docker image to ECR Repository that's created for your application.
- It creates, updates, or, configures ECS Task Definition and ECS Service to apply your configurations.

Now let's check the application status using [status](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-status) command:

```bash
coldbrew status
```

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-status.gif?v=1" width="800">

It gives you much more details about your application and its related AWS resources.

_*Again, it will take several minutes until all AWS resources get fully provisioned and become active. But, the next deploys will be much faster, typically within a minute._

#### Deploying Again

After the first deploy, whenever you have changes in your code or configurations, you can re-deploy them again using the same [deploy](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-deploy) command. In this example, I made a simple change in `.cpu` attributes in the configuration file: from `0.1` tom `0.2`.

```bash
coldbrew deploy
```

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-deploy-2.gif?v=1" width="800">

You will notice that **coldbrew-cli** did not create a new AWS resources this time because they were already created during the first deploy run. **coldbrew-cli** always tries to minimize the actual AWS changes by analyzing and comparing the current status and the desired status.

_*Note that not all configuration changes will be applied immediately. See [this](https://github.com/coldbrewcloud/coldbrew-cli/wiki/Configuration-Changes-and-Their-Effects) for more details._

Let's run [status](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-status) command again:

```bash
coldbrew status
```

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-status-2.gif?v=1" width="800">

## Testing the Application

Once your bot application is up and running, you will see the bot user become online on Slack channels. You can test the bot by mentioning him and saying something.

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-screen1.png?v=2">

## Cleaning Up

When you no long need to run your application in AWS, you can use [delete](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-delete) command to clean up all AWS resources that were used to run your application.

```bash
coldbrew delete
```

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-delete.gif?v=1" width="800">

_*Note that cleanining up can take several minutes to finish to make sure all AWS resources are properly deleted or updated._

And, if you need to delete the cluster too, you can [cluster-delete](https://github.com/coldbrewcloud/coldbrew-cli/wiki/CLI-Command:-cluster-delete) command.

```bash
coldbrew cluster-delete tutorial
```

<img src="https://raw.githubusercontent.com/coldbrewcloud/assets/master/coldbrew-cli/tutorial-echo-slack-bot-cluster-delete.gif?v=1" width="800">

_*For the same reason, cluster delete can take long to finish._

---

That's it for the tutorial. 

See [Documentations](https://github.com/coldbrewcloud/coldbrew-cli/wiki) for more details on **coldbrew-cli**, or, check out other tutorials:

- [Running a scalable WordPress website on AWS](https://github.com/coldbrewcloud/tutorial-wordpress)
- [Running a Go application on AWS](https://github.com/coldbrewcloud/tutorial-echo)
- [Running a Node.JS application on AWS](https://github.com/coldbrewcloud/tutorial-nodejs)
