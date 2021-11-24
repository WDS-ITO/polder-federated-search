# POLDER Federated Search

## A federated search app for and by the polar data community

### Covered data repositories
- [Arctic Data Center]( https://arcticdata.io/)
- [Netherlands Polar Data Center](https://npdc.nl)
- [BCO-DMO](https://www.bco-dmo.org/)
- Todo: If I'm doing global DataOne searches, what else might I cover?
- [Australian Antarctic Data Centre](https://data.aad.gov.au/)
- [National Snow and Ice Data Center](https://nsidc.org])

### Architecture
This tool uses docker to manage the different services that it depends on. One of those is [Gleaner](https://gleaner.io).

The web app itself that hosts the UI and does the searches is built using [Flask](https://flask.palletsprojects.com), which is a Python web framework. I chose Python because DataOne has [client libraries](https://dataone-python.readthedocs.io/en/latest/#python-libraries-for-software-developers) written in Python, and Python has good support for RDF and SPARQL operations with [RDFLib](https://rdflib.dev/).

### Deployment

#### Docker
Assuming that you're starting from **this directory**:
To build the Docker image for this app, run `docker image build . `. Then, you can run it using `docker run`.

A pre-built image is on Docker Hub as nein09/polder-federated-search.

#### Helm/Kubernetes

1. Install helm (on OS X, something like `brew install helm`), or visit the [Helm website](http://helm.sh) for instructions.
1. This Helm chart uses an ingress controller, which you need to install, like this:
    ```helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
  ```
1. Assuming that you're starting from **this directory**, run `helm install polder ./helm` ; the `polder` can be replaced with whatever you want.
1. The cluster will take a few minutes to spin up. In addition to downloading all these Docker images and starting the web app, it does the following:
  1. Starts a Blazegraph Triplestore and creates a namespace in it
  1. Starts a Minio / S3 storage system
  1. Initializes Gleaner
  1. Kicks off a Gleaner index of the data repositories we want to get from there
  1. Writes the resulting indexed metadata to Blazegraph so we can search it
1. If you're using Docker desktop for all this, you can visit [http://localhost](http://localhost) and see it running!

#### Setup
Assuming that you're starting from **this directory**:

1. Install [Docker](https://docker.com)

1. `cd docker`
1. `docker-compose up -d`
1. Go to your [local Blazegraph instance](http://localhost:9999/blazegraph/#namespaces) and add a new namespace - this is because the default one does not come with a full text index. Name it `polder` (or whatever you want, but you will need to change the GLEANER_ENDPOINT_URL environment variable if you don't name it that), and check the text boxes after "Full text index" and "Enable geospatial".
1. If you want to try queries out on Blazegraph directly, be sure to click 'Use' next to your new namespace after you create it.
1. `docker-compose --profile setup up -d` in order to start all of the necessary services and set up Gleaner for indexing.

For development, you can do `flask run` from this directory, or follow the directions in the Deployment section above to run the app with Docker.

If you want to run the pre-built image web app as part of docker-compose, instead of your local Docker image or by itself with `flask run`, you can do `docker-compose --profile web up -d`.

#### Doing a crawl
Assuming that you're starting from **this directory**:

1. `cd docker`
1. `curl -O https://schema.org/version/latest/schemaorg-current-https.jsonld`
1. `docker-compose --profile crawl up -d`

