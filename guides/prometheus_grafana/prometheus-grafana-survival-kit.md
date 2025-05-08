# Prometheus Grafana Survival Kit.

## Resources.

1a. CloudBeesTV Youtube Tutorial(How to Monitor Jenkins With Grafana and Prometheus): https://www.youtube.com/watch?v=3H9eNIf9KZs

1b. Tutorial gist file: https://gist.github.com/darinpope/1c8422fb7512411760ccb2827d82613f

2. Prometheus Docker image: https://hub.docker.com/r/prom/prometheus

3. Grafana Docker image: https://hub.docker.com/r/grafana/grafana

## Create Host EC2 Virtual Machines(VM) And Make Relevant Installations.

1. Proceed to create two EC2 virtual machines(one for Prometheus, and one for Grafana). I recommend that each stays on a separate VM.
2. Install docker on the VMs(refer to my other(free) github [guide on installing Docker on AWS EC2 machines](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/ec2_docker-management.md)).
3. Optional: Install Nginx, then use it to get an SSl certificate for a domain/sub-domain(e.g. jenkins.mydomain.com or grafana.mydomain.com) - reverse-proxying, and directing to your domain/sub-domain. Also refer to my free [EC2 Nginx guide](https://github.com/Okpainmo/aws-cloud-survival-kit/blob/main/tutorials/ec2_nginx-management.md) for handling all of that.

## Installing/Running Prometheus with Docker.

1. Pull in the prometheus docker image:

```bash
docker pull prom/prometheus
```

2. Create/Open a custom .yml config file for prometheus

```bash
sudo vim /home/ubuntu/prometheus-config.yml
```

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
    - targets: ['13.51.172.10:8080'] # ensure to add the correct IP of the server running the jenkins setup you wish to track
```

4. Run the container with reference to the config file and test on port `9090`:

```bash
sudo docker run -d -p 9090:9090 -v /home/ubuntu/prometheus-config.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

Stop whenever you wish:

```bash
sudo docker stop container-id
```

```bash
sudo docker rm container-id
```

5. Create a system service to persist the prometheus Docker process.

To persist and auto-start a Docker container using `systemd`, you can create a custom service unit. Below is a **standard `systemd` service file** example for running a Docker container (like Prometheus):

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

### Using the Prometheus UI.

1. Accessing the added yaml configuration.

> **Status > configuration**

----

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

> Update with more content - especially the UI aspects
