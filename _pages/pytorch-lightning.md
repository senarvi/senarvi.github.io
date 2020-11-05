---
title: "PyTorch Lightning"
permalink: /notes/pytorch-lightning/
author_profile: true
---

## Data Pipeline

### [LightningDataModule](https://github.com/PyTorchLightning/pytorch-lightning/blob/master/pytorch_lightning/core/datamodule.py)

* Contains data loaders for training, validation, and test sets
* As an example, see the [PASCAL VOC data module](https://github.com/PyTorchLightning/pytorch-lightning-bolts/blob/master/pl_bolts/datamodules/vocdetection_datamodule.py)

### [DataLoader](https://github.com/pytorch/pytorch/blob/master/torch/utils/data/dataloader.py)

* An iterator for sampling batches from a dataset
* Loads the data using multiple workers
* Takes care of batching and shuffling the samples

### [Dataset](https://github.com/pytorch/pytorch/blob/master/torch/utils/data/dataset.py)

* Defines how to read and process the data samples
* Often downloads the data package over the network first
* [torchvision](https://github.com/pytorch/vision/tree/master/torchvision/datasets) contains a collection of standard datasets
* As an example, see the [PASCAL VOC dataset](https://github.com/pytorch/vision/blob/master/torchvision/datasets/voc.py)
* The [constructor](https://github.com/pytorch/vision/blob/release/0.8.0/torchvision/datasets/voc.py#L157) downloads and extracts the tar archive
* The [__getitem__ method](https://github.com/pytorch/vision/blob/release/0.8.0/torchvision/datasets/voc.py#L202) parses the XML annotations, applies any transforms, and returns a data point

### Transform

* Defines how a data point is transformed
* Can be used to normalize the samples, or augment the dataset with more samples
* [torchvision](https://github.com/pytorch/vision/tree/master/torchvision/transforms) contains a collection of standard transforms
* For example, [Resize](https://github.com/pytorch/vision/blob/release/0.8.0/torchvision/transforms/transforms.py#L232), [RandomCrop](https://github.com/pytorch/vision/blob/release/0.8.0/torchvision/transforms/transforms.py#L484)
