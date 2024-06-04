es8.13启动时的各种默认生成的密码token等


```
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  t447iY*ScLtdaEHTzWs*

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  8563f6c2189208cfdc6ec7234008986dd2e0c9d42fbc576a355f68266bab4807

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEzLjQiLCJhZHIiOlsiMTkyLjE2OC4yNi4xNjA6OTIwMSJdLCJmZ3IiOiI4NTYzZjZjMjE4OTIwOGNmZGM2ZWM3MjM0MDA4OTg2ZGQyZTBjOWQ0MmZiYzU3NmEzNTVmNjgyNjZiYWI0ODA3Iiwia2V5IjoiR0RzVjRZOEJOZ1cxUDJuVG5xV2c6dDFsVHF2NXhRYXlQTEM0OUhJakYzQSJ9

ℹ️ Configure other nodes to join this cluster:
• Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEzLjQiLCJhZHIiOlsiMTkyLjE2OC4yNi4xNjA6OTIwMSJdLCJmZ3IiOiI4NTYzZjZjMjE4OTIwOGNmZGM2ZWM3MjM0MDA4OTg2ZGQyZTBjOWQ0MmZiYzU3NmEzNTVmNjgyNjZiYWI0ODA3Iiwia2V5IjoiR1RzVjRZOEJOZ1cxUDJuVG5xV2c6by05NG85SzFRbFM4RElJTXhwVDl1QSJ9

  If you're running in Docker, copy the enrollment token and run:
  `docker run -e "ENROLLMENT_TOKEN=<token>" docker.elastic.co/elasticsearch/elasticsearch:8.13.4`


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━




[2024-06-04T10:30:45,455][INFO ][o.e.x.s.a.Realms         ] [node-1] license mode is [basic], currently licensed security realms are [reserved/reserved,file/default_file,native/default_native]
[2024-06-04T10:30:45,457][INFO ][o.e.l.ClusterStateLicenseService] [node-1] license [91c0748f-5043-430b-bb85-e723ad45b276] mode [basic] - valid
^C[2024-06-04T10:33:01,165][INFO ][o.e.x.m.p.NativeController] [node-1] Native controller process has stopped - no new native processes can be started



```


```
./elasticsearch-reset-password --batch --user kibana_system
New value: jn7a4u3OCF5D2DN1r0Ht


./elasticsearch-reset-password --batch --user elastic
New value:  t447iY*ScLtdaEHTzWs*

```