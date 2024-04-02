# Challenge Specific Setup Guide

### Prerequisite

0. Prerequisite. Make sure you are using an RTX compatible GPU. We recommend the OS version <tt>Ubuntu 18.04/20.04</tt> and NVIDIA driver version <tt>525.60.11</tt>. Docker-based setup is not guaranteed to work on other OS or driver versions.
We also tested the following driver version that is compatible with isaac sim:

```
470.141.03
535.113.01
```

1. Generate NVIDIA NGC API Key
   - Log in [NVIDIA NGC](https://catalog.ngc.nvidia.com/). If you do not have an account, register one and log in.
   - Generate your NGC API key. You can refer to [Generating API key](https://docs.nvidia.com/ngc/gpu-cloud/ngc-user-guide/index.html#generating-api-key).
   - Log into the NGC account on the instance
   ```bash
   docker login nvcr.io
   ```
   Type `$oauthtoken` for `Username`. Then paste your API key for `Password`. You should see `Login Succeeded`.

### Download code base
git clone from the challenge branch
```bash
git clone -b challenge https://github.com/arnold-benchmark/arnold.git
```

### move to the workspace 

```bash
cd arnold/workspace
```

### build docker image
Make sure nvidia docker is properly installed

```bash
docker build -f Dockerfile -t "arnold" .
sudo apt install vagrant
vagrant up
vagrant ssh

```

###     Download challenge data and assets
For all data and assets unzip under /vagrant
* challenge_data_train.zip is for training
* data_for_challenge_final.zip for testing phase
* data_for_challenge_val.zip for dev phase
* materials.zip for scene and object materials
* sample.zip for assets used in ARNOLD
```bash
pip install gdown
cd /vagrant
gdown https://drive.google.com/uc?id=1diLNQQcOGKEVkgOstkRbagbn_cVCKVIE
gdown https://drive.google.com/uc?id=1XKRxsByOwI_pYh09LUQ5wGLgzeGmL-KH
gdown --folder https://drive.google.com/drive/folders/1Kkoi65l_m5Vnv65P5qukWOas_t2J5tG0?usp=drive_link
gdown https://drive.google.com/uc?id=1CAT6pZfX0HqHKXU5qBdLeRZl_iY_XfOt
gdown https://drive.google.com/uc?id=1jscZWcibfVXItbY1xZxRogA6Q8j3U60C
unzip data_for_challenge_val.zip
unzip data_for_challenge_final.zip
unzip challenge_data_train.zip
unzip sample.zip
unzip materials.zip
```

###  Model Evaluation 

Please read these two evaluation scripts carefully to understand what happened:

```bash
eval_challenge_dev.sh checkpoint_dir # for dev phase
eval_challenge.sh checkpoint_dir # for testing phase
```

The first time starting the Isaac Sim takes long time, approximately 6 minutes (vary based on the device configuration).

After evaluation is done, the scripts will generate an output folder. Zip the output folder (`/root/arnold/output` or `/root/arnold/output_dev` depending on the phase) and submit it to EvalAI using command line

```
zip -r submission_dev.zip -r /root/arnold/output_dev/
zip -r submission_test.zip -r /root/arnold/output/
```
