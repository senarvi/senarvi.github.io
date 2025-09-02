---
title: "Data format is crucial for good training speed"
chart: true
---

When your training runs start to take days, you might be tempted to throw more GPUs at the problem. It's easy to underestimate the time needed for tasks that are not done on the GPU, such as reading data and performing augmentation on the fly. It's entirely possible that your GPU spends most of the training time waiting for new data. One easily overlooked decision that can have a huge impact on training speed is the format in which you store your data. I would argue that the data format is much more important than, for example, the choice of storage device (HDD or SSD).

### Shared file systems

HPC clusters use large distributed file systems that are shared by all users. These file systems are designed to provide very high throughput by striping data across multiple servers. For this to work, data must be stored in sufficiently large files. Additionally, when accessing a large number of small files, the metadata server can become the bottleneck.

The naive way to store data is having each sample in a separate file, or even multiple files per sample. For example, an object detection dataset might contain the image files and one text file per image that contains the object annotations. Using multiple GPUs that each use multiple threads to simultaneously access those files may choke the file system.

### Streaming data formats

For training machine learning models, you want to convert your dataset from individual files into a more efficient format. There are several alternatives that store data in larger chunks called *shards* and take advantage of the fact that higher throughput can be achieved by reading data sequentially. Shuffling training samples is achieved by reading the shards in a random order and reading the samples within each shard in a random order.

These libraries also support streaming data from inexpensive cloud storage during training, which, surprisingly, is often fast enough. This development has been accelerated by the demand for training large language models on massive corpora.

**WebDataset** packs data samples into tar archives. The [library](https://github.com/webdataset/webdataset), developed by Thomas Breuel at NVIDIA since 2020, provides a "fluid" interface for defining preprocessing pipelines.

```python
import webdataset as wds
pil_dataset = wds.WebDataset(url).shuffle(1000).decode("pil").to_tuple("png", "json")
```

The biggest issue I had with the WebDataset library was that getting distributed training to work required some hacking. Much of the functionality was later implemented in the `DataPipe` class in the [TorchData](https://github.com/pytorch/data) project, which I found to work decently. In 2022, Breuel [stated](https://github.com/webdataset/webdataset/issues/194) that all WebDataset functionality will be merged into TorchData. However, in 2024, the PyTorch team announced they would deprecate `DataPipe`. All of this has made me question the future of this data format.

**MosaicML**, a machine learning platform that was acquired by Databricks in 2023, uses a data format called Mosaic Data Shard (MDS). First, the `MDSWriter` class is used to create the shards. At this stage, the data is already processed as much as possible, so that reading will be fast. During training, data will be read using the `StreamingDataset` class. The data streaming library can be installed also separately without the full MosaicML platform.

The **LitData** project, started in 2023 by the Lightning team, has been developed quite actively. Its data format is similar to MDS, and it provides a `StreamingDataset` class that is very similar to that of MosaicML. I've found this to work very well with PyTorch Lightning.

All of these libraries implement data parallelism by distributing different shards to different GPUs. When we launch the training process on each GPU, the assumption is that all processes receive the same number of batches during one epoch. The libraries have mechanisms to enforce that. However, some more complex data augmentation techniques may consume a random number of data samples, which can be tricky to implement with a streaming data reader.

Take, for example, mosaic augmentation, which produces a mosaic image from four images. Typically, this is implemented by randomly sampling the additional three images from the dataset. Randomly sampling images would defeat the purpose of the streaming data reader. Instead, you want to sample the images from the shard currently loaded by the process. Alternatively, you could just consume the next three images in sequence. But if you perform the augmentation randomly, different GPUs may receive a different number of batches, causing the processes to hang when they try to synchronize at the end of the epoch.

### SquashFS

SquashFS is a compressed read-only file system, commonly used to package Linux distributions in a "live CD". If you only want to avoid cluttering a file system with many small files, and your dataset is not so large that you need to stream it directly from cloud storage during training, you can simply package the data into a single SquashFS file. It's possible to mount the contents of a SquashFS file to a directory, so that training tools can access the samples without any code changes. SquashFS allows fast random access to the data samples, so any existing data processors, such as mosaic augmentation, work without problems.

For example, in an HPC clusters, you would typically run your training tool from a Singularity container. You can use the `--bind` argument to mount a SquashFS file in the container.

```sh
singularity exec \
  --bind train.squashfs:/data/train:image-src=/ \
  --bind val.squashfs:/data/val:image-src=/ \
  --bind test.squashfs:/data/test:image-src=/ \
  container-image.sif \
  python train.py /data
```

Another situation where it's helpful to package data into a SquashFS file is when you want to train directly from object storage. I tested mounting a Google Cloud Storage bucket in a virtual machine using Cloud Storage FUSE and training an object detection model by reading directly from the mounted bucket. Training directly from the storage bucket was almost as fast as training on data first copied to a network-attached HDD, as long as the data was packaged in a SquashFS file. On the other hand, reading individual files from a storage bucket using Cloud Storage FUSE is extremely slow.

<div style="max-width:600px; margin: 2em 0;">
  <canvas id="trainingTimeChart"></canvas>
</div>

<script>
document.addEventListener("DOMContentLoaded", function() {
  const ctx = document.getElementById('trainingTimeChart').getContext('2d');
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: ['Cloud Storage FUSE', 'Cloud Storage FUSE + SquashFS', 'Network-attached HDD'],
      datasets: [{
        label: 'Hours',
        data: [12.098, 2.163, 2.154],
        backgroundColor: ['#4e79a7', '#f28e2b', '#18a330ff']
      }]
    },
    options: {
      indexAxis: 'y',
      responsive: true,
      plugins: {
        legend: { display: false },
        title: { display: true, text: 'Training Time' }
      },
      scales: {
        x: {
          beginAtZero: true,
          title: { display: false },
          ticks: { display: false },
          grid: { display: false }
        }
      }
    }
  });
});
</script>
