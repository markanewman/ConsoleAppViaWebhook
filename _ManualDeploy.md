# Enviormental Setup

Before you can do anything, there are some amount of prerequsits involved.
They are listed below.

* Install [Docker][docker] desktop
    * You will need to setup a free account on [Docker][docker] to download the software
* Install [Python][python]
    * Install the non-core packages
```{cmd}
pip install applicationinsights
pip install azure-storage-blob
pip install azure-storage-queue
```
* Make your _primary_ [Azure][azure] resources
    * You will need to setup a trial account on [Azure][azure] to be able to do this. 
    * [Storage account][storage]
	* [Application Insights][appinsights]
* Make a `todo` and a `done` [blob][blob] container in your [storage account][storage]
* Make a `todo` and a `done` [queue][queue] in your [storage account][storage]
	
# Worker

1. Open a command console
2. Change to the `./Worker` directory
3. Make sure there is a `secrets.ini`.
   The _xxx_ are replaced by the values found in your _primary_ [Azure][azure] resources
```{txt}
[AZURE]
iKey = xxx
AccountName = xxx
AccountKey = xxx
```
4. Place a file in the `todo` [blob][blob] container
5. Place a message containing the file name in the `todo` [queue][queue]
6. Run `python worker.py`

## Expected Result

1. There should be a new file in the `done` [blob][blob] container.
   The content should be "0--" then the content of the origional file
2. There should be a new message in the `done` [queue][queue] with the file name as the content
3. The worker should terminate automaticaly once the `todo` [queue][queue] is empty
4. The origional file should remain in the [blob][blob] container
5. After a 5-10 min delay, there should be some notices in the [Application Insights][appinsights] detailing the process

# Docker

1. Make sure the steps in **Worker** have been followed and can run sucessfuly
2. Open a command console
3. Change to the repo's root directory
    * [Docker][docker] docker has a 'context'.
	  All files that you want to use need to be under it.
	  In this case the `./Docker` and `./Worker` are siblings so make the 'context' their parent
4. Make the Docker Image
```{shell}
docker build -t python-via-webhook -f ./Docker/Dockerfile  .
```

## Docker cheet sheet

List/remove an image

```{shell}
docker images -a
docker rmi {{Image ID}}
```

Make a new image

```{shell}
cd "C:\Users\{{user}}\Desktop\Docker"
docker build -t my-python-app ./app
```

list/stop/remove a container

```{shell}
docker ps -a
docker kill {{Container ID}}
docker rm {{Container ID}}
```

Shell into a container that has [Python][python] installed

```{shell}
docker run -it python:3
```


-----------
https://blogs.technet.microsoft.com/uktechnet/2018/04/04/run-your-python-script-on-demand-with-azure-container-instances-and-azure-logic-apps/


# Data Flow

As part of the initial setup, the service owner needs to provide the service user a URL and an access token.

1. The client makes a [multipart form post][multipart] (HTTP POST) to the [Webhook][webhook] URL; passing in an optional callback URL.
2. The [Webhook][webhook] writes the information to [blob storage][blob], places a message in the [queue][queue], and returns 202(accepted) with location where the result will eventualy be placed.
3. The [web aware][webaware] console app on the [VM][vm] monitors the [queue][queue].
    1. When a message is received, the file information is retrieved from [blob storage][blob] and copied localy to the [VM][vm].
	2. The origional console app is called via shell exec; passing in a file name corsponding to the data sent in the [multipart form post][multipart] as well as a location to place all data that should be sent back.
	3. The [web aware][webaware] console app logs the aproate things and writes the output file back to [blob storage][blob].
4. If applicable, the [Webhook][webhook] informs the client via the optional callback URL (HTTP GET) that processing is done.

[appinsights]: https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview
[azure]: https://azure.microsoft.com
[docker]: https://docs.docker.com/get-started/
[python]: https://docs.python.org/3/
[blob]: https://azure.microsoft.com/en-us/services/storage/blobs
[queue]: https://azure.microsoft.com/en-us/services/storage/queues/
[storage]: https://docs.microsoft.com/en-us/azure/storage/
[vm]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows
[webaware]: https://en.oxforddictionaries.com/definition/web_aware
[webhook]: https://en.wikipedia.org/wiki/Webhook