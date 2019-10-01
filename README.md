#  Foundations Atlas Tutorial
<p align="center">
  <img src='images/atlas_logo.png' width=70%>
</p>


# What is Atlas?

Atlas was built by our machine learning engineers to dramatically reduce the model development time, from the experimentation workflow to production.

Here are some of the core features:

**1. Experiment Management & Tracking:**

Tag experiments and easily track hyperparameters, metrics, and artifacts such as images, GIFs, and audio clips in a web-based GUI to track the performance of your models

**2. Job queuing & scheduling:**

Launch and queue thousands of experiment variations to fully utilize your system resources

**3. Collaboration & Bookkeeping:**

Keep a journal of thoughts, ideas, and comments on projects

**4. Reproducibility:**

Maintain an audit trail of every single experiment you run, complete with code and any saved items

<p float="left">
  <img src="images/tracking.png" width=40% />
  <img src="images/queue.png" width=40% /> 
</p>

# Start Guide

**Prerequisites**

1. Docker version >18.09 (Docker installation: <a target="_blank" href="https://docs.docker.com/docker-for-mac/install/"> Mac</a>
 | <a target="_blank" href="https://docs.docker.com/docker-for-windows/install/"> Windows</a>
2. Python >3.6 (<a target="_blank" href="https://www.anaconda.com/distribution/">Anaconda installation</a>)
3. \>5GB of free machine storage
4. The atlas_ce_installer.py file (Download after signup <a target="_blank" href="https://www.atlas.dessa.com/">here</a>)


**Steps**

See <a target="_blank" href="https://dessa-atlas-community-docs.readthedocs-hosted.com/en/latest/ce-quickstart-guide/">Atlas documentation</a>
  

<details>
  <summary>FAQ: How to upgrade an older version of Atlas?</summary>
<br>

1. Stop atlas server using `atlas-server stop`
2. Remove docker images related to Atlas in your terminal `docker images | grep atlas-ce | awk '{print $3}' | xargs docker rmi -f`
3. Remove the environment where you installed the Atlas or pip uninstall the Atlas `conda env remove -n your_env_name`

-------------------------------------------------------------------------------------------------------------------------
</details>

# Image Segmentation

This tutorial demonstrates how to make use of the features of Foundations Atlas. Note that **any machine learning job can be run in Atlas without modification.** However, with minimal changes to the code we can take advantage of Atlas features that will enable us to:

* view artifacts such as plots and tensorboard logs, alongside model performance metrics
* launch many training jobs at once
* organize model experiments more systematically


## Data and Problem

The dataset that will be used for this tutorial is the <a target="_blank" href="https://www.robots.ox.ac.uk/~vgg/data/pets/">Oxford-IIIT Pet Dataset</a>, created by Parkhi *et al*. The dataset consists of images, their corresponding labels, and pixel-wise masks. The masks are basically labels for each pixel. Each pixel is given one of three categories :

* Class 1 : Pixel belonging to the pet.
* Class 2 : Pixel bordering the pet.
* Class 3 : None of the above/ Surrounding pixel.

Download the processed data [here](https://dl-shareable.s3.amazonaws.com/train_data.npz).

<img src='images/data.png' width=70%>

## Clone the Tutorial

Clone this repository and make it your current directory by running:
```bash
git clone https://github.com/dessa-public/Image-segmentation-tutorial.git
```

## Start Atlas

Activate the conda environment in which Foundations Atlas is installed. Then run `atlas-server start` in a new tab terminal. Validate that the GUI has been started by accessing it at <a target="_blank" href="http://localhost:5555/projects">http://localhost:5555/projects</a>.


## Build Docker Image

The motivation of building customized image is to avoid reinstall packages listed in the requirements.txt repeatedly. 
```bash
docker build . --tag image_seg:atlas
```


## Enabling Atlas Features

You are provided with the following python scripts:

main.py: A main script which prepares data, trains an U-net model, then evaluates the model on the test set.

To enable Atlas features, we only to need to make a few changes. Firstly add the following line to the top of main.py:

```python
import foundations
```

## Logging Metrics and Parameters
In the end of main.py, add:

```python
foundations.log_metric('train_accuracy', float(train_acc))

foundations.log_metric('val_accuracy', float(val_acc))

```

## Saving Artifacts

Line 108 in main.py:
```python
foundations.save_artifact(f"sample_{name}.png", key=f"sample_{name}")

foundations.save_artifact('trained_model.h5', key='trained_model')
```

## TensorBoard Integration 
Line 210 in main.py:
```python
foundations.set_tensorboard_logdir('tflogs')
```

## Configuration
job_config.yaml
```python
# Project config #
project_name: 'Image-segmentation-tutorial'
log_level: INFO

# Worker config #
# Additional definition for the worker can be found here: https://docker-py.readthedocs.io/en/stable/containers.html

num_gpus: 0

worker:
  image: image_seg:atlas
```

* To run a single job with Atlas, type the following in the terminal:
```python
foundations submit scheduler . main.py
```
Make sure your current directory is `Image-segmentation-tutorial`.

* To run multiple experiments, run:
```python
python hyperparameter_search.py
```

```python
import os
import numpy as np
import foundations

NUM_JOBS = 10

def generate_params():

    hyper_params = {'batch_size': int(np.random.choice([8, 16, 32, 64])),
                    'epochs': int(np.random.choice([10, 20, 30, 50])),
                    'learning_rate': np.random.choice([0.1, 0.01, 0.001, 0.0001]),
                    'decoder_neurons': [np.random.randint(16, 512), np.random.randint(16, 512),
                                        np.random.randint(16, 512), np.random.randint(16, 512)],
                    }
    return hyper_params


for job_ in range(NUM_JOBS):
    print(f"packaging job {job_}")
    hyper_params = generate_params()
    foundations.submit(scheduler_config='scheduler', job_dir='.', command='foundations_main_to_run.py', params=hyper_params,
                       stream_job_logs=False)
```
Line 27, replace hyp
```python
hyper_params = foundations.load_parameters()
```



