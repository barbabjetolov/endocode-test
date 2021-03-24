# endocode-test

This repo contains a small application, `http-service`, written in Golang that serves few HTTP endpoints, and the scripts (jenkinsfile + helm chart + terraform plan) needed to automate its deployment in an existing kubernetes cluster.

# http-service

`http-service` is a simple golang application that serves HTTP requests. 

## Requirements

#### OS requirements
* macOs, Linux

#### with docker
* docker
#### local install
* go 1.13 or newer
* make

## Building and running

#### docker
To build a docker image simply run `make docker`: this will build and run a container that contains the application. By default it listens from the 8080 port.

#### local install
To compile the source code run `make compile`, it will take care of the dependencies.  To run it, launch `make run`. As noted before, by default the service listens from the 8080 port, but it can be changed by setting the environment variable `LISTENING_PORT`.

## Usage

The service listens accepts two endpoints:

* **GET /helloworld** - returns "Hello Stranger". It accepts one query parameter, `name`. If set, it returns "Hello $name", sliced by camel case
* **GET /versionz** - returns a JSON with the hash of the latest commit and the project name

For example, to call the first endpoint, you'd do: 
```shell
    curl localhost:8080/helloworld
```

and the answer will be:
```
    Hello Stranger
```

To use the query funcionality, use this request
```shell
    curl localhost:8080/helloworld?name=MarcoRossi
```

and the answer will be:
```
    Hello Marco Rossi
```

The last endpoint can be contacted with
```
    curl localhost:8080/versionz
```

and the answer will be:
```json
    {
        "git_commit": "2d23bd462aa5523a0bdcd272d4958700e3cc6eac",
        "project_name": "http-service"
    }
```

## Deploy
This service can be deployed in an existing kubernetes cluster using the provided jenkinsfile. The cluster IP has to be set manually in the `main.tf` file.

The helm chart contains templates to create a service, a deployment and an nginx-ingress that allows communication with the service. 

## Logging
The repo has a terraform file, `main_ekf.tf`, that contains instructions to install an EKF stack in the kubernetes cluster. It's based on the official elastic helm charts for [Kibana](https://github.com/elastic/helm-charts/tree/6.5.2-alpha1/kibana) and [Elasticsearch](https://github.com/elastic/helm-charts/blob/6.5.2-alpha1/elasticsearch/README.md), plus custom charts for Fluentd, `fluentd-chart` and for an ingress that exposes the Kibana UI, by default on the address `http://kibana.int`. To access it, add the line `$CLUSTER_IP kibana.int` in the host `/etc/hosts` file. 

## Server stub
A configuraton file to generate a server stub is provided, `openapi.yaml`. To generate the stub run the commands:

```shell
    git clone https://github.com/openapitools/openapi-generator
    cd openapi-generator
    mvn clean package
    java -jar modules/openapi-generator-cli/target/openapi-generator-cli.jar generate \
    -i openapi.yaml \
    -g {LANGUAGE} \
    -o /var/tmp/php_api_client
```

There are several `LANGUAGE` choices, check the [openapi-generator github page](https://github.com/OpenAPITools/openapi-generator) for a complete list.