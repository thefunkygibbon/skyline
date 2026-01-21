# Grafana with Prometheus for Check Point Skyline

A monitoring solution for Checkpoint physical and virtual appliances.
this docker compose script will install and configure an instance of Prometheus and Grafana as seperate containers, including required configuration parameters and dashboard templates.
The script will pull the official images, and use the Grafana provisioning feature to automatically configure the Grafana datasource and dashboard template.
[Prometheus](https://prometheus.io/), [Grafana](http://grafana.org/), [Skyline](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk178566)

**Note - this connfiguration does not use TLS to secure any communication. If you want to use secured the communication, please refer to the Skyline-TLS repository which requires a few more steps to deploy. [Skyline-TLS](https://github.com/deangoldhill/skyline-TLS)

## Install Docker

Install Docker-CE and docker-compose:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## Install Containers

Clone this repository on your Docker host, cd into the skyline directory and run compose up:

```bash
git clone https://github.com/thefunkygibbon/skyline.git
cd skyline

docker-compose up -d
```



## Setup Grafana

Navigate to `http://<DOCKERHOST>:3000` and login with user ***admin*** password ***admin***. You can change the password from within the Grafana portal, or you can modify the credentials in the compose file.


```

Grafana is preconfigured with dashboards and Prometheus as the default data source:

* Name: SKYLINE_PROMETHEUS
* Type: Prometheus
* Url: http://prometheus:9090

There are 4 Skyline dashboards

***Machines Overview***

Machines with top CPU / Memory utilisations
Vital stastics of selected (filter) machine.


***Single Machine***
In-depth information and statistics for a single machine,
Hardware resource consumption, version info, policy info, network throughput, connections,Drops, errors etc.

***Scalable Platforms***
As above but for Maestro platforms

***VSX***
VSX server and indavidual virtual system stastics


```

## Setup Skyline on Check Point machines

Create a configuration payload file on each Check Point machine with the following contents, and import it.
Modify <DOCKERHOST> with the IP or hostname of your docker host.

Navigate to the '/var/tmp' directory, create a new file called 'payload.json' in which you will paste the content from the next step.

```bash

cd /var/tmp
vi payload.json

```

Paste the following into the file you just created and then save the file. (Modify <DOCKERHOST> with the IP or hostname of your docker host.)

```bash

 {
    "enabled": true,
    "export-targets": {"add": [
        {
            "enabled": true,
            "type": "prometheus-remote-write",
            "url": "http://<DOCKERHOST>:9090/api/v1/write"
        }
    ]}
} 

```
Execute the payload execution command

```bash

sklnctl export --set "$(cat payload.json)"

```
