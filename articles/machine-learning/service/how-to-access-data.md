---
title: Access data in datastores / blobs for training
titleSuffix: Azure Machine Learning service
description: Learn how to use datastores to access blob data storage during training with Azure Machine Learning service
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: minxia
author: mx-iao
ms.reviewer: sgilley
ms.date: 02/25/2019
ms.custom: seodec18


---

# Access data from your datastores

 In Azure Machine Learning service, datastores are compute location-independent mechanisms to access storage without requiring changes to your source code. Whether you write training code to take a path as a parameter, or provide a datastore directly to an estimator, Azure Machine Learning workflows ensure your datastore locations are accessible, and made available to your compute context.

This how-to shows examples of the following tasks:
* [Choose a datastore](#access)
* [Get data](#get)
* [Upload and download data to datastores](#up-and-down)
* [Access datastore during training](#train)

## Prerequisites

To use datastores, you first need a [workspace](concept-azure-machine-learning-architecture.md#workspace).

Start by either [creating a new workspace](setup-create-workspace.md#sdk) or retrieving an existing one:

```Python
import azureml.core
from azureml.core import Workspace, Datastore

ws = Workspace.from_config()
```

<a name="access"></a>

## Choose a datastore

You can use the default datastore or bring your own.

### Use the default datastore in your workspace

 Each workspace has a registered, default datastore that you can use right away.

To get the workspace's default datastore:

```Python
ds = ws.get_default_datastore()
```

### Register your own datastore with the workspace

If you have existing Azure Storage, you can register it as a datastore on your workspace.   All the register methods are on the [`Datastore`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py) class and have the form register_azure_*. 

The following examples show you to register an Azure Blob Container or an Azure File Share as a datastore.

+ For an **Azure Blob Container Datastore**, use [`register_azure_blob-container()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py)

  ```Python
  ds = Datastore.register_azure_blob_container(workspace=ws, 
                                               datastore_name='your datastore name', 
                                               container_name='your azure blob container name',
                                               account_name='your storage account name', 
                                               account_key='your storage account key',
                                               create_if_not_exists=True)
  ```

+ For an **Azure File Share Datastore**, use [`register_azure_file_share()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py#register-azure-file-share-workspace--datastore-name--file-share-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false-). For example: 
  ```Python
  ds = Datastore.register_azure_file_share(workspace=ws, 
                                           datastore_name='your datastore name', 
                                           file_share_name='your file share name',
                                           account_name='your storage account name', 
                                           account_key='your storage account key',
                                           create_if_not_exists=True)
  ```

<a name="get"></a>

## Find & define datastores

To get a specified datastore registered in the current workspace, use [`get()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py#get-workspace--datastore-name-) :

```Python
#get named datastore from current workspace
ds = Datastore.get(ws, datastore_name='your datastore name')
```

To get a list of all datastores in a given workspace, use this code:

```Python
#list all datastores registered in current workspace
datastores = ws.datastores
for name, ds in datastores.items():
    print(name, ds.datastore_type)
```

To define a different default datastore for the current workspace, use [`set_default_datastore()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#set-default-datastore-name-):

```Python
#define default datastore for current workspace
ws.set_default_datastore('your datastore name')
```

<a name="up-and-down"></a>
## Upload & download data
The [`upload()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.azure_storage_datastore.azureblobdatastore?view=azure-ml-py#download-target-path--prefix-none--overwrite-false--show-progress-true-) and [`download()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.azure_storage_datastore.azureblobdatastore?view=azure-ml-py#download-target-path--prefix-none--overwrite-false--show-progress-true-) methods described in the following examples are specific to and operate identically for the [AzureBlobDatastore](https://docs.microsoft.com/python/api/azureml-core/azureml.data.azure_storage_datastore.azureblobdatastore?view=azure-ml-py) and [AzureFileDatastore](https://docs.microsoft.com/python/api/azureml-core/azureml.data.azure_storage_datastore.azurefiledatastore?view=azure-ml-py) classes.

### Upload

 Upload either a directory or individual files to the datastore using the Python SDK.

To upload a directory to a datastore `ds`:

```Python
import azureml.data
from azureml.data.azure_storage_datastore import AzureFileDatastore, AzureBlobDatastore

ds.upload(src_dir='your source directory',
          target_path='your target path',
          overwrite=True,
          show_progress=True)
```

`target_path` specifies the location in the file share (or blob container) to upload. It defaults to `None`, in which case the data gets uploaded to root. `overwrite=True` will overwrite any existing data at `target_path`.

Or upload a list of individual files to the datastore via the datastore's `upload_files()` method.

### Download
Similarly, download data from a datastore to your local file system.

```Python
ds.download(target_path='your target path',
            prefix='your prefix',
            show_progress=True)
```

`target_path` is the location of the local directory to download the data to. To specify a path to the folder in the file share (or blob container) to download, provide that path to `prefix`. If `prefix` is `None`, all the contents of your file share (or blob container) will get downloaded.

<a name="train"></a>
## Access datastores during training

Once you make your datastore available on the compute target, you can access it during training runs (for example, training or validation data) by simply passing the path to it as a parameter in your training script.

The following table lists the  [`DataReference`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.data_reference.datareference?view=azure-ml-py) methods that tell the compute target how to use the datastore during runs.

Way|Method|Description|
----|-----|--------
Mount| [`as_mount()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.data_reference.datareference?view=azure-ml-py#as-mount--)| Use to mount the datastore on the compute target.
Download|[`as_download()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.data_reference.datareference?view=azure-ml-py#as-download-path-on-compute-none--overwrite-false-)|Use to download the contents of your datastore to the location specified by `path_on_compute`. <br> For  training run context, this download happens before the run.
Upload|[`as_upload()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.data_reference.datareference?view=azure-ml-py#as-upload-path-on-compute-none--overwrite-false-)| Use to upload a file from the location specified by `path_on_compute` to your datastore. <br> For training run context, this upload happens after your run.

 ```Python
import azureml.data
from azureml.data.data_reference import DataReference

ds.as_mount()
ds.as_download(path_on_compute='your path on compute')
ds.as_upload(path_on_compute='yourfilename')
```  

To reference a specific folder or file in your datastore and make it available on the compute target, use the datastore's [`path()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.data_reference.datareference?view=azure-ml-py#path-path-none--data-reference-name-none-) function.

```Python
#download the contents of the `./bar` directory in ds to the compute target
ds.path('./bar').as_download()
```

> [!NOTE]
> Any `ds` or `ds.path` object resolves to an environment variable name of the format `"$AZUREML_DATAREFERENCE_XXXX"` whose value represents the mount/download path on the target compute. The datastore path on the target compute might not be the same as the execution path for the training script.

### Compute context and datastore type matrix

The following matrix displays the available data access functionalities for the different compute context and datastore scenarios. The term "Pipeline" in this matrix refers to the ability to use datastores as an input or output in [Azure Machine Learning Pipelines](https://docs.microsoft.com/azure/machine-learning/service/concept-ml-pipelines).

||Local Compute|Azure Machine Learning Compute|Data Transfer|Databricks|HDInsight|Azure Batch|Azure DataLake Analytics|Virtual Machines|
-|--|-----------|----------|---------|-----|--------------|---------|---------|
|AzureBlobDatastore|[`as_download()`] [`as_upload()`]|[`as_mount()`]<br> [`as_download()`] [`as_upload()`] <br> Pipeline|Pipeline|Pipeline|[`as_download()`] <br> [`as_upload()`]|Pipeline||[`as_download()`] <br> [`as_upload()`]|
|AzureFileDatastore|[`as_download()`] [`as_upload()`]|[`as_mount()`]<br> [`as_download()`] [`as_upload()`] Pipeline |||[`as_download()`] [`as_upload()`]|||[`as_download()`] [`as_upload()`]|
|AzureDataLakeDatastore|||Pipeline|Pipeline|||Pipeline||
|AzureDataLakeGen2Datastore|||Pipeline||||||
|AzureDataPostgresSqlDatastore|||Pipeline||||||
|AzureSqlDatabaseDataDatastore|||Pipeline||||||


> [!NOTE]
> There may be scenarios in which highly iterative, large data processes run faster using [`as_download()`] instead of [`as_mount()`]; this can be validated experimentally.

### Examples 

The following code examples are specific to the [`Estimator`](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py) class for accessing your datastore during training.

This code creates an estimator using the training script, `train.py`, from the indicated source directory using the parameters defined in `script_params`, all on the specified compute target.

```Python
from azureml.train.estimator import Estimator

script_params = {
    '--data_dir': ds.as_mount()
}

est = Estimator(source_directory='your code directory',
                entry_script='train.py',
                script_params=script_params,
                compute_target=compute_target
                )
```

You can also pass in a list of datastores to the Estimator constructor `inputs` parameter to mount or copy to/from your datastore(s). This code example:
* Downloads all the contents in datastore `ds1` to the compute target before your training script `train.py` is run
* Downloads the folder `'./foo'` in datastore `ds2` to the compute target before `train.py` is run
* Uploads the file `'./bar.pkl'` from the compute target up to the datastore `ds3` after your script has run

```Python
est = Estimator(source_directory='your code directory',
                compute_target=compute_target,
                entry_script='train.py',
                inputs=[ds1.as_download(), ds2.path('./foo').as_download(), ds3.as_upload(path_on_compute='./bar.pkl')])
```

## Next steps

* [Train a model](how-to-train-ml-models.md)

* [Deploy a model](how-to-deploy-and-where.md)
