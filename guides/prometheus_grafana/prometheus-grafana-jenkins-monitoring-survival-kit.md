# Prometheus-Grafana Jenkins Monitoring Survival Kit.

- Prometheus is a telemetry pipeline tool that connects to your services(e.g. Jenkins(CI/CD) controllers, servers, and databases), to extracts certain relevant information from them.

- Grafana on the other hands, is a monitoring/observability/visualization tool. It helps with rendering tracked data on powerful dashboards that can be customized to suit different use-cases. With Grafana, you can visualize your data from different sources - all in one place.

As can already be inferred, the goal of this guide, is to track the performance of a Jenkins(CI/CD) controller - with much ease and flexibility on a Grafana dashboard with the aid of Prometheus tracking.

You will learn completely and perfectly how to implement the above as stated, while also getting solid hands-on experience to help with implementing telemetry pipelines on any running system service(e.g. databases and servers) using Prometheus and Grafana.

> P.S: A more complete telemetry pipeline for Jenkins will be a setup like:
>
> 1. Metrics: Prometheus
>
> 2. Logs: Loki(Grafana), or ELK(Elasticsearch, Logstash, Kibana)
>
> 3. Traces: Tempo(Grafana), Jaeger, or OpenTelemetry Collector

## Resources.

1a. CloudBeesTV Youtube Tutorial(How to Monitor Jenkins With Grafana and Prometheus): https://www.youtube.com/watch?v=3H9eNIf9KZs

1b. Tutorial gist file: https://gist.github.com/darinpope/1c8422fb7512411760ccb2827d82613f

2. Prometheus Docker image: https://hub.docker.com/r/prom/prometheus

3. Grafana Docker image: https://hub.docker.com/r/grafana/grafana

4. Setting up Jenkins on AWS EC2: https://github.com/Okpainmo/jenkins-prometheus-grafana-ci-cd-survival-kit/blob/main/guides/jenkins_core/1_installing_jenkins_on_aws_ec2.md

5. Jenkins Plug-ins to install:

  - Prometheus Metrics: https://plugins.jenkins.io/prometheus/

  - CloudBees Disk Usage Simple: https://plugins.jenkins.io/cloudbees-disk-usage-simple/

## Create Host EC2 Virtual Machines(VMs) And Make Relevant Installations.

For this guide, we'll need 3 EC2 VMs.

> - The first machine should be the one running your Jenkins controller - I recommend a direct Jenkins installation without Docker.
> - The second machine should run Grafana using Docker.
> - The third should run Prometheus - as well using Docker.

1. So proceed to create three EC2 virtual machines(one for the Jenkins controller, one for Prometheus, and one for Grafana). I recommend that each stays on a separate VM.

2. Install docker on the Prometheus and Grafana VMs(refer to my other(free) github [guide on installing Docker on AWS EC2 machines](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/ec2_docker-installation.md)).

3. Optional: Install Nginx, then use it to get an SSl certificate for domains/sub-domains(e.g. jenkins.mydomain.com, prometheus.mydomain.com and grafana.mydomain.com). With that in place, you're able to handle reverse-proxying, and directing to your domains/sub-domains while avoiding to open any extra port - hence keeping your VM as safe as possible. Also refer to my free [EC2 Nginx guide](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/ec2_nginx-management.md) for handling all of that.

## Setting Up and Running The Jenkins Controller.

I have a free and complete [guide on how to make a standard Jenkins installation on AWS EC2](https://github.com/Okpainmo/jenkins-prometheus-grafana-ci-cd-survival-kit/blob/main/guides/jenkins_core/1_installing_jenkins_on_aws_ec2.md). Explore that guide, and you'll be good.

> The Nginx guide above, will help you with everything you need to implement reverse proxy-ing, and setting up SSL certified custom domains.
>
> **P.S: This guide from this point on, assumes that you've not implemented reverse-proxying and a custom domain setup for your systems. Hence I worked directly with the VM access ports. So feel free to update with the proper domains/subdomains whenever/wherever necessary**.

## Installing/Running Prometheus with Docker.

1. Pull in the prometheus docker image:

```bash
docker pull prom/prometheus
```

2. Create/Open a custom .yml config file for prometheus

```bash
sudo vim /home/ubuntu/prometheus-config.yml
```

or

```bash
sudo nano /home/ubuntu/prometheus-config.yml
```

3. Add the following config to it: 

> The config is currently suited for observing Jenkins using Prometheus and Grafana

```yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
  scrape_timeout:      10s

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'jenkins'
    metrics_path: /prometheus/ # ensure to always add the trailing "/" as seen here - after '/prometheus'
    static_configs:
    - targets: ['jenkins-vm-public-ip:8080'] # ensure to add the correct IP of the server running the jenkins controller that you wish to track
```

4. Run the container with reference to the config file and test on port `9090`:

```bash
sudo docker run -d -p 9090:9090 -v /home/ubuntu/prometheus-config.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

Stop  and remove whenever you wish:

```bash
sudo docker stop container-id
```

```bash
sudo docker rm container-id
```

5. Create a system service to persist the prometheus Docker process.

To persist and auto-start a Docker container using `systemd`, you can create a custom service unit. Below is a **standard `systemd` service file** example for running a Docker container(like Prometheus):

- Step 1: Create the service file

```bash
sudo vim /etc/systemd/system/prometheus-docker.service
```

- Step 2: Add the following content for the `prometheus-docker` service

```ini
[Unit]
Description=Prometheus Docker Container
After=docker.service
Requires=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker run --rm \
  --name prometheus \
  -p 9090:9090 \
  -v /home/ubuntu/prometheus-config.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

ExecStop=/usr/bin/docker stop prometheus

[Install]
WantedBy=multi-user.target
```

**Explanation**

- `--rm`: Automatically removes the container when it stops (you can remove this if you prefer it to persist).
- `Restart=always`: Ensures it auto-restarts on reboot or crash.
- `ExecStart`: The command to start your Docker container.
- `ExecStop`: Stops the running container cleanly.
- `Requires=docker.service`: Ensures Docker is started first.

Step 3: Reload the systemd daemon

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```

Step 4: Enable and start the service

```bash
sudo systemctl enable prometheus-docker
sudo systemctl start prometheus-docker
# sudo systemctl restart prometheus-docker
```

Step 5: Check the service with status:

```bash
sudo systemctl status prometheus-docker
```

View service logs whenever necessary(e.g. when you need to debug):

```bash
journalctl -u prometheus-docker.service -f
```

## Installing/Running Grafana with Docker.

1. Pull in the grafana docker image:

```bash
docker pull grafana/grafana
```

2. Run the image and test on port `3000`:

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

Stop and/or remove the container whenever you wish:

```bash
sudo docker stop container-id
```

```bash
sudo docker rm container-id
```

3. Create a system service to persist the Grafana Docker process.

Similar to the above implementation for Prometheus, to persist 
and auto-start a Docker container using `systemd`, 
we'll need to create a custom service unit. 

- Step 1: Create the service file

```bash
sudo vim /etc/systemd/system/grafana-docker.service
```

- Step 2: Add the following content for the `grafana-docker` service

```ini
[Unit]
Description=Grafana Docker Container
After=docker.service
Requires=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker run --rm \
  --name grafana \
  -p 3000:3000 \
  grafana/grafana

ExecStop=/usr/bin/docker stop grafana

[Install]
WantedBy=multi-user.target
```

**Explanation**

- `--rm`: Automatically removes the container when it stops (you can remove this if you prefer it to persist).
- `Restart=always`: Ensures it auto-restarts on reboot or crash.
- `ExecStart`: The command to start your Docker container.
- `ExecStop`: Stops the running container cleanly.
- `Requires=docker.service`: Ensures Docker is started first.

Step 3: Reload the systemd daemon

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```

Step 4: Enable and start the service

```bash
sudo systemctl enable grafana-docker
sudo systemctl start grafana-docker
# sudo systemctl restart grafana-docker
```

Step 5: Check the service with status:

```bash
sudo systemctl status grafana-docker
```

View service logs whenever necessary:

```bash
journalctl -u grafana-docker.service -f
```

## Accessing Your Jenkins Connection To Prometheus.

As can be seen in our Prometheus yaml configuration;

```bash
...

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'jenkins'
    metrics_path: /prometheus/ # ensure to always add the trailing "/" as seen here - after '/prometheus'
    static_configs:
    - targets: ['jenkins-vm-public-ip:8080'] # ensure to add the correct IP of the server running the jenkins controller that you wish to track
```

We have the above part, that reveals our prometheus connection to the Jenkins controller.

> The above config tells us that our prometheus data-tracking for the Jenkins VM is on the `/prometheus/` path. Hence, accessing the below address, should reveal a stream of data that that verifies that our tracking is already in progress.

```bash
  http://jenkins-vm-public-ip:8080/prometheus/ # always ensure to add the trailing slash
```

## Configuring Prometheus Tracking on Jenkins.

Head to **Jenkins > Manage Jenkins > System > (Scroll to the) Prometheus Section**, and update the Prometheus configuration like below:

- Path: Leave default - 'prometheus'.
- Collecting metrics period in seconds: set to 5.
- Collect disk usage: ensure this is checked.
- Save and exit.

## Access the Prometheus UI.

1. Visiting the prometheus UI: 

```bash
http://prometheus-vm-ip/9090
```

2. Accessing the added yaml configuration.

> **Status(on the top nav) > configuration**

3. Exploring tracked metrics directly in the Prometheus UI.

  - Look for vertical 3-dots icon within the 'Enter expression...'(search) bar, click on it, then select "explore metrics".
  - From the list of displayed metrics, select(copy the name/text of) any that you wish to see a response for, then close that tab.
  - paste the copied metric name into the 'Enter expression...'(search) bar, then click the 'Execute' button.

  E.g. **`jenkins_executor_count_value`** returns **`jenkins_executor_count_value{instance="13.53.150.111:8080", job="jenkins"}	2`**.

## Access the Grafana UI.

- Access the Grafana UI here: 

```bash
http://grafana-vm-ip/3000
```

- Enter the default access credentials.

  - Username: admin
  - Password: admin

- Create you own password on the next provided screen.

### Setting Up a Grafana Data Source.

- Inside the Grafana UI, on the left-side menu, select "Data sources".
- On the top-right of the next UI, select "Add new data source".
- Next, select Prometheus.
- Prometheus configuration:

  - Give it a befitting name - especially in case you'll be having multiple Prometheus data-sources.
  - Connection: add link to the Prometheus server.

    ```bash
    http://prometheus-vm-ip/9090
    ```
  - Save and test the connection - if everything is great, you should see a confirmation notification.
- Return to your data sources, and you should now see you new addition.

### Importing a Grafana Dashboard.

- Visit the Grafana Dashboard website - `https://grafana.com/grafana/dashboards`.
- Scroll slightly below, and use the search bar to search for any suitable 'Jenkins' dashboard.
  
  > I used 'Jenkins: Performance and Health Overview - by Haryan'. 
  >
  > Id: '9964'

- Click on the one you prefer, then copy it's 'Id'.
- Return to the Grafana UI, and click on the plus icon at the top-right. Then select 'import dashboard'.
- Paste in the dashboard Id, then click 'load' button for 'Id' import.
- On the next UI, name your dashboard, then select the data source you created/connected earlier and finally, click the relevant button, and finish up.
- At the top-right of your new dashboard - which should now be showing(even though data in-flow might not happen immediately), select the option with a clock icon(the tracking period setup), then set to "Last 30 days".
- Also at the top-right of your new dashboard select the option with a refresh icon(the refresh rate option), then set to "5s".

> **There you have it - a complete setup for tracking your Jenkins CI/CD controller with Prometheus and Grafana**.


