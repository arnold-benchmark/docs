# Setup for Challenge

### Prerequisite

0. Prerequisite. Make sure you are using an RTX compatible GPU. We recommend the OS version <tt>Ubuntu 18.04/20.04</tt> and NVIDIA driver version <tt>525.60.11</tt>. Docker-based setup is not guaranteed to work on other OS or driver versions.
We also verified the following driver versions are compatible with isaac sim:

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

2. Make sure NVIDIA container is properly installed. Check [Installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

### Docker Setup

1. Download codebase: `git clone` from the `challenge` branch.
```bash
git clone -b challenge https://github.com/arnold-benchmark/arnold.git
```

2. Move to `workspace` and build docker image.

```bash
cd arnold/workspace

docker build -f Dockerfile -t "arnold" .

sudo apt install vagrant
vagrant up
vagrant ssh
```

### Download Data and Assets
You can download [data and assets](https://drive.google.com/drive/folders/1yaEItqU9_MdFVQmkKA6qSvfXy_cPnKGA?usp=drive_link) from web browser or CLI. Unzip the downloaded `zip` files at `/vagrant` folder.
* `data_for_challenge_train.zip` for training
* `data_for_challenge_val.zip` for the `dev` phase
* `data_for_challenge_final.zip` for the `test` phase
* `materials.zip` for scene and object materials
* `sample.zip` for assets used in ARNOLD

```bash
# for example, download from CLI
pip install gdown

cd /vagrant
# data_for_challenge_train.zip
gdown https://drive.google.com/uc?id=1gxSW3fFhGghJUpf_jiAy3iR_zrPy_MlJ
# data_for_challenge_val.zip
gdown https://drive.google.com/uc?id=1diLNQQcOGKEVkgOstkRbagbn_cVCKVIE
# data_for_challenge_final.zip
gdown https://drive.google.com/uc?id=1XKRxsByOwI_pYh09LUQ5wGLgzeGmL-KH
# materials.zip
gdown https://drive.google.com/uc?id=1CAT6pZfX0HqHKXU5qBdLeRZl_iY_XfOt
# sample.zip
gdown https://drive.google.com/uc?id=1jscZWcibfVXItbY1xZxRogA6Q8j3U60C

unzip data_for_challenge_train.zip
unzip data_for_challenge_val.zip
unzip data_for_challenge_final.zip
unzip materials.zip
unzip sample.zip
```

###  Model Evaluation 

Please read these two evaluation scripts carefully to understand what happened:

```bash
cd ~/arnold/
bash eval_challenge_dev.sh ${checkpoint_file} # for dev phase
bash eval_challenge.sh ${checkpoint_file} # for test phase
```

To reproduce the results using the baseline checkpoint: 
```bash
gdown https://drive.google.com/uc?id=1YuADlTFJZQc3AefULmzhZ9PrCWVjsEU2
bash eval_challenge_dev.sh peract_multi_clip_best.pth
bash eval_challenge.sh peract_multi_clip_best.pth
```

The first time starting the Isaac Sim takes long time, approximately 6 minutes (varying to the device configuration).

After evaluation is done, the scripts will generate an output folder. Zip the output folder (`/root/arnold/output` or `/root/arnold/output_dev`, depending on the phase) and submit it to `EvalAI` with CLI.

```bash
zip -r submission_dev.zip -r /root/arnold/output_dev/
zip -r submission_test.zip -r /root/arnold/output/

evalai challenge 2266 phase 4500 submit --file submission_dev.zip --large
evalai challenge 2266 phase 4501 submit --file submission_test.zip --large
```
