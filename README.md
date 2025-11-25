#--------------------------------------------------------#
# ReadMe: Prometheus Guidebook
#--------------------------------------------------------#

A minimal, self-contained setup for running Prometheus + Alertmanager using Docker.  


----------------------------------------------------------
# Repository Structure
```
Prometheus-Guidebook/
├── prometheus.yml     # main config file
├── alerts.yml         # alert rules
└── alertmanager.yml   # alertmanager routing + notification config
```
----------------------------------------------------------


----------------------------------------------------------
# The Overall Workflow 
Prometheus scrapes targets according to prometheus.yml.
Alert rules in alerts.yml evaluate those metrics.
When an alert fires, Prometheus sends it to Alertmanager.
Alertmanager applies routing rules from alertmanager.yml.
You receive an email notification (or Slack, etc., if configured).
----------------------------------------------------------


----------------------------------------------------------
# Run the Stack

1. Create a Docker network for containers to communicate
docker network create my_monitoring_network

2. Start Prometheus
docker run -d \
  --name prom \                     # container name
  --network my_monitoring_network \ # Join the shared network so targets resolve by name
  -p 9090:9090 \                    # Expose Prometheus UI at http://localhost:9090
  -v "$(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml:ro" \  # the main config
  -v "$(pwd)/alerts.yml:/etc/prometheus/alerts.yml:ro" \ # the alert rule file
  prom/prometheus:latest            # base image

3. Start AlertManager
docker run -d \
  --name alertmanager \             # Name used by Prometheus when sending alerts
  --network my_monitoring_network \
  -p 9093:9093 \                    # Expose Alertmanager UI at http://localhost:9093
  -v "$(pwd)/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro" \
  prom/alertmanager:latest

4. Check Alert Status
Prometheus UI → http://localhost:9090/alerts   # Shows which alerts are active/firing
Alertmanager UI → http://localhost:9093        # Shows how alerts are routed/grouped
----------------------------------------------------------
