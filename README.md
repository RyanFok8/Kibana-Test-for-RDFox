# Using Kibana to visualise RDFox performance regression test
**remember to add lisence
## Requirements

* [Docker Engine](https://docs.docker.com/get-docker/) version **18.06.0** or newer
* [Docker Compose](https://docs.docker.com/compose/install/) version **1.28.0** or newer (including [Compose V2][compose-v2])
* 1.5 GB of RAM

## Setting up
#### Windows

If you are using the legacy Hyper-V mode of _Docker Desktop for Windows_, ensure [File Sharing](https://docs.docker.com/desktop/settings/windows/#file-sharing) is
enabled for the `C:` drive.

#### macOS

The default configuration of _Docker Desktop for Mac_ allows mounting files from `/Users/`, `/Volume/`, `/private/`,
`/tmp` and `/var/folders` exclusively. Make sure the repository is cloned in one of those locations or follow the
instructions from the [documentation](https://docs.docker.com/desktop/settings/mac/#file-sharing) to add more locations.

Make sure your Docker Desktop is opened and clone this repository onto the Docker host that will run the stack with the command below:

```sh
git clone https://github.com/deviantony/docker-elk.git
```

Open `elasticsearch.yml` and switch the value of `xpack.license.self_generated.type` setting from `trial` to `basic` to disable paid features.

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

You can change your password by clicking the icon located on the top left corner on the home page.

### Quit/Cleanup

To stop Elasticsearch and Kibana:
```sh
docker-compose stop
```

To shutdown the stack and remove all persisted data:
```sh
docker-compose down
```

## Getting started
**Note index patterns has been renamed to data views
#### To create a visualisation from scratch

1. Run the regression test, and generate a `.jsonl` file by including 'jsonl' in the '-a'
2. Upload the `.jsonl` file in the **Get started by adding integrations** section on the home page and press import
3. Create an Index name to identify the source file
4. 


#### If you have a saved dashboard `.ndjson` and a different `.jsonl` file
1. Open the sidebar, scroll to the bottom, and click **Management**
2. In the **Kibana** section, choose **Saved objects**
3. Import the dashboard you want






## Miscellaneous
### Formatting numbers
If you want to change the presentation of numbers in visualisation, e.g., to the nearest integer, to 3 decimal places...
1. Open the sidebar, scroll to the bottom, and click **Management**
2. In the **Kibana** section, click data view
3. Choose the specifc data view, and a specific key by clicking the pencil icon on the right
4. Toggle the **Set format**, and choose the appropriate **Format** and **Numeral.js format pattern** (Default is 3dp)
5. Save the settings

### Importing and exporting Dashboards (`.ndjson`)

### Combining data views/index patterns
1. Go to **Management**, click **Data Views** in **Kibana**
2. Click **Create data view**
3. To combine two or more data views/index patterns, seperate them using comma, e.g., `index_1, index_2, index_3` in the index pattern row

### Double checking the basic license
1. Go to **Management** and scroll to the bottom
2. Click on **License Management** and it should show **Your Basic license is active**

