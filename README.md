
# Batch Scoring Deep Learning Models With Kubernetes

## Overview
In this repository, we use the scenario of applying style transfer onto a video (collection of images). This architecture can be generalized for any batch scoring with deep learning scenario.

## Design
![Reference Architecture Diagram](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/batchscoringdl-aks-architecture-diagram.PNG)

The above architecture works as follows:
1. Upload a video file to storage.
2. The video file will trigger Logic App to spin up an ACI.
3. The ACI will first preprocess the video file by splitting the video into individual images and extracting the audio file.
4. The ACI will then add all images to the Service Bus queue.
5. The AKS cluster is continuously polling the Service Bus queue - as soon as any images are in the queue, it will pull it off the queue and apply style transfer to the image.
6. The ACI will wait until all images have completed processing by the AKS cluster - when it is done, it will download all processed images and stitch it back together into a video with the audio file.
7. The ACI will upload the final video back to storage.

### What is Neural Style Transfer 

| Style image: | Input/content video: | Output video: | 
|--------|--------|---------|
| <img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/style_image.jpg" width="300"> | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video.mp4 "Input Video") *click to view video* | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video.mp4 "Output Video") *click to view* |

## Prerequsites

Local/Working Machine:
- Ubuntu >=16.04LTS (not tested on Mac or Windows)
- [NVIDIA Drivers on GPU enabled machine](https://linuxconfig.org/how-to-install-the-nvidia-drivers-on-ubuntu-18-04-bionic-beaver-linux) [Additional ref: [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker)]
- [Conda >=4.5.4](https://conda.io/docs/)
- [Docker >=1.0](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1) 
- [AzCopy >=7.0.0](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)
- [Azure CLI >=2.0](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)

Accounts:
- [Dockerhub account](https://hub.docker.com/)
- [Azure Subscription](https://azure.microsoft.com/en-us/free/) (with a [quota](https://docs.microsoft.com/en-us/azure/azure-supportability/resource-manager-core-quotas-request) for GPU-enabled VMs)

While it is not required, it is also useful to use the [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) to inspect your storage account.e `az cli` installed and logged into

## Setup

1. Clone the repo `git clone <repo-name>`
2. `cd` into the repo
3. Setup your conda env using the _environment.yml_ file `conda env create -f environment.yml` - this will create a conda environment called __batchscoringdl__
4. Activate your environment `source activate batchscoringdl`
5. Log in to Azure using the __az cli__ `az login`
6. Log in to Docker using the docker cli `docker login`

#### Run the Kubernetes Dashboard
Run `az aks browse -n $aks_cluster -g $resource_group` in your terminal so that you can use the Kubernetes Dashboard. If you're not able to access the dashboard, follow the instructions [here](https://blog.tekspace.io/kubernetes-dashboard-remote-access/).

## Steps
Run throught the following notebooks:
1. [Test the Style Transfer Script](/00_test_neural_style_transfer.ipynb)
2. [Setup Azure - Resource group, Storage, Service Bus](/01_setup_azure.ipynb).
3. [Apply Style Transfer Locally](./02_style_transfer_locally.ipynb)
4. [Apply Style Transfer on AKS](./03_style_transfer_on_aks.ipynb)
5. [Deploy Logic Apps](./04_deploy_logic_app.ipynb)

## Clean up
To clean up your working directory, you can run the `clean_up.sh` script that comes with this repo. This will remove all temporary directories that were generated as well as any configuration (such as Dockerfiles) that were created during the tutorials. This script will _not_ remove the `.env` file. 

To clean up your Azure resources, you can simply delete the resource group that all your resources were deployed into. This can be done in the `az cli` using the command `az group delete --name <name-of-your-resource-group>`, or in the portal. If you want to keep certain resources, you can also use the `az cli` or the Azure portal to cherry pick the ones you want to deprovision.

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
