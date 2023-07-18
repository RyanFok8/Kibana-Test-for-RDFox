# Using Kibana to visualise RDFox performance regression test

### Setting up

Clone this repository onto the Docker host that will run the stack with the command below:

```sh
git clone https://github.com/deviantony/docker-elk.git
```

Open "elasticsearch.yml" and switch the value of `xpack.license.self_generated.type` setting from `trial` to `basic` to disable paid features.

Then, open the folder in the terminal and initialize the Elasticsearch users and groups required by docker-elk by executing the command:

```sh
docker-compose up setup
```

If the setup completed without an error, start the other stack components:

```sh
docker-compose up
```

Give Kibana about a minute to initialize, then access the Kibana web UI by opening <http://localhost:5601> in a web
browser and use the following (default) credentials to log in:

* user: *elastic*
* password: *changeme*
