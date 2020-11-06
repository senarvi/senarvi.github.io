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
* The [`\_\_getitem\_\_`](https://github.com/pytorch/vision/blob/release/0.8.0/torchvision/datasets/voc.py#L202) method parses the XML annotations, applies any transforms, and returns a data point

### Transform

* Defines how a data point is transformed
* Can be used to normalize the samples, or augment the dataset with more samples
* [torchvision](https://github.com/pytorch/vision/tree/master/torchvision/transforms) contains a collection of standard transforms
* For example, [`Resize`](https://github.com/pytorch/vision/blob/release/0.8.0/torchvision/transforms/transforms.py#L232), [`RandomCrop`](https://github.com/pytorch/vision/blob/release/0.8.0/torchvision/transforms/transforms.py#L484)

## Training

### [Trainer](https://github.com/PyTorchLightning/pytorch-lightning/blob/master/pytorch_lightning/trainer/trainer.py)

* The [`fit`](https://github.com/PyTorchLightning/pytorch-lightning/blob/release/1.0.x/pytorch_lightning/trainer/trainer.py#L386) method trains a model on given data
* Training is controlled using constructor arguments such as `max_epochs`, `precision`, `gradient_clip_val`, `gpus`, `accelerator`
* The constructor arguments can be added to an argparse parser using the [add_argparse_args](https://github.com/PyTorchLightning/pytorch-lightning/blob/release/1.0.x/pytorch_lightning/utilities/argparse_utils.py#L137) method and used to construct a Trainer using the [from_argparse_args](https://github.com/PyTorchLightning/pytorch-lightning/blob/release/1.0.x/pytorch_lightning/utilities/argparse_utils.py#L21) method:

```python
parser = ArgumentParser()
parser = Trainer.add_argparse_args(parser)
args = parser.parse_args()
trainer = Trainer.from_argparse_args(args)
```

* By default, model checkpoints are written to `lightning_logs/version_N/checkpoints/`, where N is the experiment version
* To resume training from a model checkpoint, pass a file name or URL in the `resume_from_checkpoint` argument
