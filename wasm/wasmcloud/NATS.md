### Echo Service
- echopool
```shell
nats reply my.echo --echo --queue echopool
nats req my.echo 'Yes hello this is dog'
```
- reply with a curl
	- sets up a NATS reply subscriber that listens for messages on the subject `my.weather`. When it receives a message, it executes the `curl` command to fetch current weather information in a specific format from `wttr.in` and sends this information back as a reply.
```shell
nats reply my.weather --command 'curl -s https://wttr.in/?format=3'
nats req my.weather --replies=3 'Give me local weather please!'
```

### NATS Subscription Comparison to Serverless Function
- a nats reply subscriber (a message topic subscription callback function listening to messages on the topic)

