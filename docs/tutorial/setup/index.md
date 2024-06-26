# Setup

There are two setup approaches: **docker-based** and **conda-based**. We recommend the docker-based approach as it wraps everything up and is friendly to users.

## Docker-based

0. Prerequisite. We recommend the OS version <tt>Ubuntu 18.04/20.04</tt> and NVIDIA driver version <tt>525.60.11</tt>. Other driver versions are probably compatible (*e.g.*, <tt>470.141.03</tt>, <tt>535.113.01</tt>) but not guaranteed.
1. Generate NVIDIA NGC API Key
   - Log in [NVIDIA NGC](https://catalog.ngc.nvidia.com/). If you do not have an account, register one and log in.
   - Generate your NGC API key. You can refer to [Generating API key](https://docs.nvidia.com/ngc/gpu-cloud/ngc-user-guide/index.html#generating-api-key).
   - Log into the NGC account on the instance
   ```bash
   docker login nvcr.io
   ```
   Type `$oauthtoken` for `Username`. Then paste your API key for `Password`. You should see `Login Succeeded`.
2. Make sure NVIDIA container is properly installed. Check [Installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).
3. Build the docker image inside the workspace.
   ```bash
   git clone https://github.com/arnold-benchmark/arnold.git
   cd arnold/workspace
   docker build -f Dockerfile -t "arnold" .
   ```
4. Build vagrant in the workspace. This might take a long time if you are using the image from the dockerHub instead of building it locally. You can use a system monitor to stay checking the process.

   It is also possible that `Vagrantfile` contains wrong paths for your `nvidia_icd.json` and `nvidia_layers.json`. Make sure they are not empty. For example, you should check the two paths: `/etc/vulkan/icd.d/nvidia_icd.json` and `/usr/share/vulkan/icd.d/nvidia_icd.json`, one of which would always exist.

   ```ruby
   # if jsons exist in /etc (default)
   '-v', '/etc/vulkan/icd.d/nvidia_icd.json:/etc/vulkan/icd.d/nvidia_icd.json',
   '-v', '/etc/vulkan/implicit_layer.d/nvidia_layers.json:/etc/vulkan/implicit_layer.d/nvidia_layers.json',

   # if jsons exist in /usr, modify the two lines in Vagrantfile
   '-v', '/usr/share/vulkan/icd.d/nvidia_icd.json:/etc/vulkan/icd.d/nvidia_icd.json',
   '-v', '/usr/share/vulkan/implicit_layer.d/nvidia_layers.json:/etc/vulkan/implicit_layer.d/nvidia_layers.json',
   ```
   Check the above paths according to your system. After that, build vagrant.
   ```bash
   vagrant up
   ```
5. After `vagrant up` finishes, run `vagrant ssh` and you are ready to go. Enjoy the full GUI experiment with docker and <tt>Isaac Sim</tt>. The docker environment provides a wide range of development tools as well. For more details check [readme](./docker_readme.md). Notably, you need to use `/isaac-sim/python.sh` to run `python`.

## Conda-based

As a backup solution, we also introduce a conda-based setup.

1. Manually download [NVIDIA Omniverse](https://www.nvidia.com/en-us/omniverse/download/) and install.
2. Open the `NVIDIA Omniverse` platform and install `Isaac Sim 2022.1.1` (other versions are not guaranteed to work) in `Library`.
3. Clone the [code repo](https://github.com/arnold-benchmark/arnold).
   ```bash
   git clone git@github.com:arnold-benchmark/arnold.git
   cd arnold
   ```
4. Create a conda environment.
   ```bash
   conda env create -f conda_env.yaml
   conda activate arnold
   ```
5. Install `clip`:
   ```bash
   pip install git+https://github.com/openai/CLIP.git
   ```
6. Install point cloud engine:
   ```bash
   cd utils
   python setup.py build_ext --inplace
   cd ..
   ```
7. Link the libraries and toolkits of `Isaac Sim`:
   ```bash
   source ${Isaac_Sim_Root}/setup_conda_env.sh
   # e.g., source ~/.local/share/ov/pkg/isaac_sim-2022.1.1/setup_conda_env.sh
   ```
8. You are ready to run scripts. In the activated conda environment, you can directly use `python`, in contrast to `/isaac-sim/python.sh` in docker.

## Common

Data, assets, and checkpoints are common items for both setup approaches.
- Download [data and assets](https://drive.google.com/drive/folders/1yaEItqU9_MdFVQmkKA6qSvfXy_cPnKGA?usp=sharing). If you are docker-based, put them in the `vagrant` workspace, *e.g.*, `workspace/data/pour_water`, `workspace/materials`, `workspace/sample`. If you are conda-based, make sure the `materials` and `sample` are in the same folder. After preparation, check the `data_root` and `asset_root` in `configs/default.yaml` to ensure valid links. For example, `data_root/pour_water`, `asset_root/materials` and `asset_root/sample` are valid.
- (Optional) You can download pre-trained model checkpoints from [here](https://drive.google.com/drive/folders/1yaEItqU9_MdFVQmkKA6qSvfXy_cPnKGA). Considering the performance, we provide two checkpoints of multi-task PerAct, with and without an additional state head, respectively. To evaluate the checkpoints, you need to put them in the directory `${output_root}/${task}/train_${model}_${lang_encoder}_${state_head}`. For example, the checkpoint `peract_multi_clip_best.pth` should be put in `${output_root}/multi/train_peract_clip_0`. For the model with an additional state head, the last number (`state_head`) should be `1`.

## Quickstart

### Sanity Check

After setup, you can run a toy example to check if <tt>Isaac Sim</tt> is working:
- For docker-based setup:
  ```bash
  cd workspace
  vagrant ssh

  /isaac-sim/python.sh /isaac-sim/standalone_examples/api/omni.isaac.franka/pick_place.py
  ```
- For conda-based setup:
  ```bash
  conda activate arnold

  source ${Isaac_Sim_Root}/setup_conda_env.sh
  # e.g., source ~/.local/share/ov/pkg/isaac_sim-2022.1.1/setup_conda_env.sh

  python ${Isaac_Sim_Root}/standalone_examples/api/omni.isaac.franka/pick_place.py
  # e.g., python ~/.local/share/ov/pkg/isaac_sim-2022.1.1/standalone_examples/api/omni.isaac.franka/pick_place.py
  ```

It may be slow for the first time to launch <tt>Isaac Sim</tt> because of the shader compilation.

### Visualization

You can replay the demonstrations and visualize them by running:
```bash
# docker-based
/isaac-sim/python.sh eval.py task=${TASK_NAME} mode=eval use_gt=[1,1] visualize=1

# conda-based
python eval.py task=${TASK_NAME} mode=eval use_gt=[1,1] visualize=1
```

With `use_gt=[1,1]`, the running does not require a pre-trained model checkpoint.

### Training
- For example, train a single-task PerAct on `PickupObject`:
  ```bash
  # docker-based
  /isaac-sim/python.sh train_peract.py task=pickup_object model=peract lang_encoder=clip mode=train batch_size=8 steps=100000

  # conda-based
  python train_peract.py task=pickup_object model=peract lang_encoder=clip mode=train batch_size=8 steps=100000
  ```
- Train a multi-task PerAct:
  ```bash
  # docker-based
  /isaac-sim/python.sh train_peract.py task=multi model=peract lang_encoder=clip mode=train batch_size=8 steps=200000

  # conda-based
  python train_peract.py task=multi model=peract lang_encoder=clip mode=train batch_size=8 steps=200000
  ```

For more details, see [Train](../train/index.md).

### Evaluation
Here we only show examples of conda-based commands. Docker-based commands substitute `python` with `/isaac-sim/python.sh`.
```bash
# checkpoint selection
python ckpt_selection.py task=${TASK_NAME} model=peract lang_encoder=clip mode=eval visualize=0

# evaluation
python eval.py task=${TASK_NAME} model=peract lang_encoder=clip mode=eval visualize=0
```

For more details, see [Eval](../eval/index.md).
