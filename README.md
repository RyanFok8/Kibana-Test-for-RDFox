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
