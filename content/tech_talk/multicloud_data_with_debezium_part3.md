---
title: "Multicloud data with Debezium - Part 3: Data Change History in BigQuery"
date: "2020-07-21"
description: "Using BigQuery in GCP as our historical data change repository"
featured_image: "/images/debezium.jpeg"
---
Welcome to Part 3 of this series on multicloud data with Debezium. For those just joining us, in [part 1]({{< ref multicloud_data_with_debezium_part1 >}}) you can learn about the high level architecture of what we will be building and get a quick introduction to Debezium as a change data capture tool.

In [Part 2]({{< ref multicloud_data_with_debezium_part2 >}}) we start building our solution, specifically the AWS side of things, where our data will originate.

We are now going to look in this 3rd and last part where we get to consume and extract value from this data from a different cloud, the Google Cloud Platform, and we will see the full solution in action!

Before we do anything, the same disclaimer you will see across all three parts: **this experiment is not production ready**, do not go about and try to implement this as is in a production setting unless you're looking for a dramatic way to find a new job.

# Prerequisites

We are now getting ready to start working with multiple clouds at the same time, so all prerequisites defined on [part 2]({{< ref multicloud_data_with_debezium_part2 >}}) are also required, plus these new requirements:

* A Google Cloud account with billing enabled.
* The gcloud command line tools. [Click here](https://cloud.google.com/sdk/install) for install instructions.
* A Google Cloud Project attached to a billing account.
* The Compute and BigQuery APIs enabled in your GCP project ([gcp getting started instructions](https://cloud.google.com/apis/docs/getting-started))
* A service account with the necessary rights for Terraform to create resources inside the GCP project

> Two important caveats: In AWS, you may run an issue with the required IAM roles for ECS on Fargate not existing the first time you run the code if you have never used the service before, if that happens, just wait a few minutes and try again. On GCP you will need to have multiple APIs enabled, including Compute Engine and BigQuery, if running the code throws any errors due to APIs disabled, simply go into the GCP Console and enable the required API.

# Some initial preparations

The focus of this article is enabling the secure connectivity between GCP and AWS, and consuming data from AWS Kafka into BigQuery, therefore I will not look into setting up the basic account details in GCP. If you need directions on how to create a project in GCP and attach it to a billing account, [this page](https://cloud.google.com/resource-manager/docs/creating-managing-projects) will give you details on how to manage projects and [here](https://cloud.google.com/billing/docs/how-to/modify-project) you can learn how to manage your project's billing.

# Architecture

![Architecture Diagram - GCP * AWS Architecture](/images/multicloud_data_with_debezium/part3_gcp+aws_architecture.svg)

# The codebase

All the code you need to deploy this solution can be found at [github.com/murillodigital/experiments](https://github.com/murillodigital/experiments) inside a directory named `multicloud_data_with_debezium`, in there you will find a subdirectory for each cloud we will be working on, today we're going to be looking into the `gcp+aws` directory. It is called GCP+AWS since we will actually be **orchestrating the creation of resources in both clouds in a specific sequence**, in order for us to establish the VPN connection.

Some components incorporated in the automation of the VPN connection were taken from [this tutorial](https://cloud.google.com/solutions/automated-network-deployment-multicloud) however the code has been considerably modified, nevertheless, if you are looking only to establish a VPN between GCP and AWS with terraform, that tutorial may come in handy.

In the `gcp+aws/` directory you will find two subdirectories: `terraform/` which holds the infrastructure as code required to spin up all GCP side components (and a handful in AWS), and a `beam/` subdirectory, where the **Apache Beam pipeline code that we will be using to ingest our data lives**. We will do a review of the code in both subdirectories next.

# Let's start with the infrastructure

## Network

Two files declare what network resources we will need to create, one defines AWS specific resources and the other those required in GCP. It is important to pay attention to the dependency chain between them, **the order in which resources are created, and the outputs those resources produce in one cloud as input to the creation of resources in the other cloud is very important**.

### network_aws.tf

In this file you will find all the AWS side required components to establish secure connectivity between both clouds. **The three primary services required to establish a VPN connection on the AWS side are a _VPN gateway_, a _Customer Gateway_ and a _VPN connection_**, in addition to those three core resources, you will need to add additional routes to your routing tables with the CIDR blocks from the other end of the tunnel, namely the GCP side in our scenario, and make sure that your routes that point to the VPN are propagated, and allow the security groups to receive connections from the GCP IP range.

### network_gcp.tf

In the case of GCP, since this is the first time we are introducing the cloud in our solution, we do need to create the core network resources necessary to build our infrastructure, together with the VPN specific resources that will connect both clouds.

In this file you will find the declarations to create the GCP Network and a single subnet, you are creating two GCP routers, one for each VPN tunnel (note, redundant VPN tunnels are established against AWS, and AWS manages both tunnels inside a single VPN connection). Two VPN tunnels are created and each peered with one of the routers I mentioned before. Last but not least, firewall rules are created, to allow the VPN traffic to flow through, as well as enable communication between the bastions on both clouds. ICMP is allowed in both directions, just to simplify validation that networks can talk to one another.

# Now let's look at the data side

## BigQuery

### bigquery.tf

There's not many surprises in terms of BigQuery, being a managed service with a pretty straightforward concept around projects, datasets and tables, the amount of resources we need to create to get data into it is minimal.

**We'll create one dataset and one table in our project. The one important aspect to note is that the _schema must be defined in this resource_.**

The reason I point that out is because throughout this experiment the schema for our data is defined in various places, we define it in the initialize.sql template you can find in the AWS side of things (see [part 2]({{< ref multicloud_data_with_debezium_part2 >}}) of this series to learn about that) as well as the Apache Beam pipeline we will be looking into soon.

Therefore, it is important to keep all those coupled data structures in sync - this would be one of the first things I'd look at in terms of automating in a further iteration of this experiment, managing coupled schemas in so many places could be a recipe for disaster.

## Apache Beam, and a lot of frustration

[Apache Beam](https://beam.apache.org/) is a pipeline execution engine for both batch and streaming data - in our case we will be using it to ingest the stream of data generated by Debezium from capturing our data changes and shipping those over to Kafka.

Beam provides official SDKs in a few languages, including Java, Python and Go. **I decided to go with the Python one**.

I really like Go but it seems to me that the Go SDK is the least mature from the pack, and since this was meant to be just an experiment, for prototyping a concept I thought Python would be the better choice than Java (not to mention it still remains one of my favorite all time languages).

**However, in hindsight I regret that choice**. I constantly struggled with the Python SDK, and the documentation available and community around it seemed still immature - perhaps it was my lack of experience with the framework, but I wasted a gigantic amount of time dealing with issues, and making sense of just unpredictable behavior that was hard to troubleshoot.

I simply could not succeed in using the native [Kafka IO module](https://beam.apache.org/releases/pydoc/2.13.0/apache_beam.io.external.kafka.html) and ended up switching to [another module from beam_nuggets.io](http://mohaseeb.com/beam-nuggets/beam_nuggets.io.kafkaio.html#beam_nuggets.io.kafkaio.KafkaConsume) that, as much as it worked, considering it's independently written by a very small set of contributors and in a very early version, I just would not recommend for any production setting.

The same story went with the native [BigQuery Module](https://beam.apache.org/releases/pydoc/2.13.0/apache_beam.io.gcp.bigquery.html), that would simply not behave as I expected and debugging was just so complicated that I, again, ended up with a solution I would not suggest for a production setting, which is writing my own DoFn and using the bigquery client from the google cloud SDK directly to write the data to BigQuery

> In a nutshell, if you choose Beam for your data pipelining efforts, I'd probably stick to the Java SDK - not because I have used it but because it can't possibly be a trickier experience than the Python one. Take this advice with a grain of salt from someone admittedly not a Beam expert, but at the pace of progress with the Python Beam SDK I experienced, becoming an expert would not be a pleasant experience.

## Google Dataflow, a bit more frustration, and a change in direction

**Originally, my goal was to use Beam so that I could take advantage of [GCP's Dataflow service](https://cloud.google.com/dataflow)**, which provides a Beam native runner for you to deploy your pipelines against a managed, highly scalable service.

**Alas, that was not a walk in the park either - so much so that I ended up pivoting to simply running my Beam pipeline as a service in a Compute Instance**. This was an experiment so bear with me for taking those shortcuts, those decisions are not ones I would (necessarily) recommend if working towards a production ready solution.

Debugging inside Dataflow was slow and complicated. Dataflow actually spins up a compute engine instance for you, and runs inside it a containerized set of services, including your Beam pipeline code.

The environments that Dataflow spins up for you comes with some packages built in, but you will still need to handle installing any other package you need deployed with your solution.

The way the Dataflow runner handles this is, it downloads all the packages in your requirements file, compresses them and puts them into a GCP storage bucket together with your compressed pipeline code --- this behavior is the one you get when you specify a `--requirements_file` when running your beam code with `--runner dataflow`.

This process was slow and painful, it always took quite some time to get the packages, compress them and push them up to the storage container, which introduced a huge delay when you're iterating and debugging.

Once the code was finally available for the runner, waiting for the install of the packages so that you could, through trial and error, catch which additional package you had to add considering the relatively obscure characteristics of the runner environment was a really slow process. I must have done over two dozen attempts just to get a bit more insights from the logs every time I did it.

There was another option I did not get to try just because I had already wasted so much time and this was just meant to be an experiment on Debezium, not Dataflow, which was to use setuptools inside the runner to get the packages in place.
 
**After many hours spent trying to debug this, and considering the experimental nature of this exercise that, after all, had a different focus, I decided to pivot and go with a good old compute engine instance running the beam pipeline as a systemd managed service**.

## The beam pipeline

You will find the beam pipeline code inside `beam/`, all the code lives in a single file (`main.py`) and is a pretty straightforward pipeline:

```python
def run(bootstrap_servers, topic, project, dataset, table):
    kafka_config = {"topic": topic,
                    "bootstrap_servers": bootstrap_servers,
                    "group_id": "debezium_consumer_group"}

    mapping_schema = {
        "sku": lambda data: data.payload.after.sku if data.payload.op != 'd' else data.payload.before.sku,
        "name": lambda data: data.payload.after.name if data.payload.op != 'd' else data.payload.before.name,
        "price": lambda data: data.payload.after.price if data.payload.op != 'd' else data.payload.before.price,
        "quantity": lambda data: data.payload.after.available if data.payload.op != 'd' else data.payload.before.available,
        "timestamp": lambda data: datetime.datetime.utcfromtimestamp(data.payload.ts_ms / 1000).isoformat(),
        "deleted": lambda data: True if data.payload.op == 'd' else False
    }

    pipeline_options = PipelineOptions(pipeline_args)
    pipeline_options.view_as(StandardOptions).streaming = True

    p = beam.Pipeline(options=pipeline_options)

    _ = (p | 'Reading messages' >> kafkaio.KafkaConsume(kafka_config)
     | 'Preparing data' >> beam.ParDo(TransformSchema(mapping_schema))
     | 'Writing data to BigQuery' >> beam.ParDo(WriteToBigQuery(dataset, project, table)))
    result = p.run()
    result.wait_until_finish()
```

I wrote two `DoFn`s, one for preparing the data for big query using a mapping schema dictionary, and another one to WriteToBigQuery. The mapping schema uses lambdas to define which attributes in the message to insert in each applicable BigQuery column, this logic is necessary for three reasons:

1) The schemas are not identical, the attribute quantity in the BigQuery table doesn't match the `available` property in the source PostgreSQL table for example
2) The structure of the Debezium generated message includes a full representation of the data before and after the event, in the cases of deletes the before will be empty, and only the after object will be present, in case of updates both objects will have data, and in case of deletes only the after object will hold information. We need to create an insert object for BigQuery with valid data in all these scenarios.
3) We are using the `ts_ms` attribute in the Debezium message as timestamp for the event change, we need to reduce the precision of that timestamp and turn it into an actual iso formatted datetime string for it to be effectively inserted in the BigQuery table.

### The pipeline service

A very simple systemd unit will run this python code as a service in the compute instance we're spinng up, and all the steps to get and run the beam code is included in the user data of the GCE instance.

#### SystemD Unit File
```systemd
[Unit]
Description=murillodigital
After=syslog.target network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/experiments/multicloud_data_with_debezium/gcp+aws/beam
ExecStart=/root/experiments/multicloud_data_with_debezium/gcp+aws/beam/bin/python \
    /root/experiments/multicloud_data_with_debezium/gcp+aws/beam/main.py \
    --bootstrap-server ${BOOTSTRAP_SERVERS} --topic ${KAFKA_TOPIC} \
    --gcp_project ${GCP_PROJECT} \
    --dataset ${DATASET} \
    --table ${TABLE}
Restart=on-failure
EnvironmentFile=/root/murillodigital.env

[Install]
WantedBy=multi-user.target
```


#### Compute Instance
```hcl
resource "google_compute_instance" "murillodigital-beam" {
  name = "murillodigital-beam"
  machine_type = "n1-standard-1"
  zone = "${var.gcp_region}-b"
  service_account {
    scopes = ["bigquery"]
  }

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  scratch_disk {
    interface = "SCSI"
  }

  network_interface {
    network = google_compute_network.gcp-network.name
    subnetwork = google_compute_subnetwork.gcp-subnet1.name
    access_config { }
  }

  metadata_startup_script = <<EOF
#!/bin/bash
apt update
apt install -y git python3-pip python3-venv
git clone https://github.com/murillodigital/experiments /root/experiments
python3 -m venv /root/experiments/multicloud_data_with_debezium/gcp+aws/beam
pushd /root/experiments/multicloud_data_with_debezium/gcp+aws/beam
. bin/activate
pip install wheel
pip install -r requirements.txt
popd
echo "BOOTSTRAP_SERVERS=${var.bootstrap_servers}" >> /root/murillodigital.env
echo "KAFKA_TOPIC=${var.kafka_topic}" >> /root/murillodigital.env
echo "GCP_PROJECT=${data.google_project.project.name}" >> /root/murillodigital.env
echo "DATASET=${var.dataset_name}" >> /root/murillodigital.env
echo "TABLE=${var.table_name}" >> /root/murillodigital.env
cp /root/experiments/multicloud_data_with_debezium/gcp+aws/beam/murillodigital.service /etc/systemd/system/murillodigital.service
chmod 644 /etc/systemd/system/murillodigital.service
systemctl start murillodigital
EOF

  tags = [
    "murillodigital-debezium"
  ]
}
```

Note how we have granted BigQuery scope to the instance, so it can perform operations against the BigQuery resource. In a production environment you will want to use a custom service account with much more specific permissions granted.

# We're now ready to take it for a test drive

Remember, you will need to have defined in your environment the necessary credentials to communicate with both clouds before we can get started, and an SSH key must be available in AWS which is the only variable required and will be used to provide access to the AWS side Bastion host.

Your current working directory should be `multicloud_data_with_debezium/` inside my [experiments repository](https://github.com/murillodigital/experiments). In there you will find the `main.tf` file that spins up the entire infrastructure across both clouds:

```hcl
provider "google" {
  region = "us-east1"
}

provider "aws" {
  region = "us-east-1"
}

module "aws" {
  source = "./aws/terraform"
  bastion_key_name = var.bastion_key_name
}

module "gcp_aws" {
  depends_on = [module.aws]
  source = "./gcp+aws/terraform"
  aws_internal_sg = module.aws.aws_internal_sg
  aws_private_route_table = module.aws.aws_private_route_table
  aws_private_subnet1 = module.aws.aws_private_subnet1
  aws_private_subnet2 = module.aws.aws_private_subnet2
  aws_public_route_table = module.aws.aws_public_route_table
  aws_vpc_id = module.aws.aws_vpc_id
  bootstrap_servers = module.aws.aws_brokers_cleartext
  kafka_topic = "debezium_multicloud.public.inventory"
}
```

All you have to do is run `terraform apply`, provide your AWS ssh key name, and wait for the magic to happen.

Give this a try and let me know how it went with a comment. Stay up to date with new experiments, join my mailing list!.

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/horizontal-slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; width:100%;}
	/* Add your own Mailchimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="https://murillodigital.us10.list-manage.com/subscribe/post?u=c12ff1afa71003663de3762cc&amp;id=4cff0f72fe" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Subscribe</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_c12ff1afa71003663de3762cc_4cff0f72fe" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>

<!--End mc_embed_signup-->