# Collecting Demos and Training Policies in RoboCasa
I've had to set up <a href="https://robocasa.ai">RoboCasa</a> to simulate manipulation tasks.

**IMAGE OR VIDEO HERE**

1. <a href="#installation">Installation</a>
2. <a href="#setting-up-the-spacemouse-pro">Setting Up the SpaceMouse Pro</a>
3. <a href="#policy-learning">Policy Learning</a>
    - <a href="#training">Training</a>
    - <a href="#logging-and-viewing-training-results">Logging and Viewing Training Results</a>

## Installation

To install RoboCasa, follow the comprehensive instructions provided in the [RoboCasa documentation](https://github.com/robocasa/robocasa). Or just run this. 
> :warning: **Make sure you are cloning the RoboCasa branches**
```bash
conda create -c conda-forge -n robocasa python=3.9
conda activate robocasa
git clone https://github.com/ARISE-Initiative/robosuite -b robocasa_v0.1
cd robosuite
pip install -e .
cd ..
git clone https://github.com/robocasa/robocasa
cd robocasa
pip install -e .
conda install -c numba numba -y
python robocasa/scripts/download_kitchen_assets.py   # Caution: Assets to be downloaded are around 5GB.
python robocasa/scripts/setup_macros.py              # Set up system variables.
```


## Setting Up the SpaceMouse Pro

You will need to install the drivers. Check out the complete documentation at [https://spacenav.sourceforge.net](https://spacenav.sourceforge.net).

```bash
git clone https://github.com/FreeSpacenav/spacenavd.git
# These dependencies may be needed:
# sudo apt-get install libxi-dev
# sudo apt-get install libXtst-dev
cd spacenavd
./configure && make && sudo make install
cd ..
git clone https://github.com/FreeSpacenav/libspnav
cd libspnav
./configure && make && sudo make install
```
Now run `spacenavd` using 
```bash
sudo spacenavd -v
```
If there is an existing `spacenavd` daemon running you will get an error: `Spacenav daemon already running (pid: 992). Aborting`. Kill this process with `kill -9 992` and start the daemon.

Confirm the install is correct by running the example in `lipsnav/example/cube`. Run `make` and `./cube`. You should be able to move the cube in 3D space with your SpaceMouse.

Finally you need spanv which you can get via pip (?) --> add link to other thing maybe...


You might need to update the `SPACEMOUSE_VENDOR_ID` and `SPACEMOUSE_PRODUCT_ID` in `robosuite/macros_private.py` and `robocasa/macros_private.py` to match your SpaceMouse. You can find these IDs using:

```bash
lsusb
```
Remember to add `0x` in front if you use the base-16 numbers.

For some SpaceMouse models, you may need to manually activate the logic by modifying the `run` method. Add `or True` to the method definition as shown below:

``` python
while True:
    d = self.device.read(13)
    if d is not None and self._enabled:

        if self.product_id == 50741 or True:
```
You may need to run as `sudo` for the `hidapi` library. To use the correct Python version (the one from your Conda environment), you will have to replace `python` with `$(which python)`. This ensures that the Python interpreter being used is the one from your Conda environment, rather than the system-wide Python.
```bash
sudo $(which python) robocasa/scripts/collect_human_demonstrations.py --device spacemouse 
```
To get started with collecting demos, first choose a task from the list provided by [robocasa](https://robocasa.ai/docs/tasks_scenes_assets/atomic_tasks.html). Use the --environment flag to indicate which task you want to collect demos for. You can also easily add your own tasks (see further down).

## Policy Learning
You have to install Robomimic. Once again, make sure to use the RoboCasa branch.
```bash
git clone https://github.com/ARISE-Initiative/robomimic -b robocasa
cd robomimic
pip install -e .
```

### Training
Each algorithm has a generator script in `scripts/config_gen`. Running this script will give you the commands for your training run(s). To start training you simply need to run the command(s) outputted. To adjust the training change the file in `config_gen` according to the [robomimic docs](https://robomimic.github.io/docs/tutorials/hyperparam_scan.html#step-3-set-hyperparameter-values). 
```bash
python robomimic/scripts/config_gen/diffusion_gen.py --name <name-to-identify-later>
```

If you want to use your own datasets you will need to convert them from the raw robosuite (RoboCasa datasets are basically robosuite datasets) into the robomimic format. There are two steps:
1. Convert to robomimic
This will convert your dataset in place.
```bash
python robomimic/scripts/conversion/convert_robosuite.py --dataset <ds-path>
```
3. Extract observations for training
This script will generate a new dataset with the suffix `_im128.hdf5` in the same directory as `--dataset`. There are a bunch of options for what observations are used. Check out `robomimic/scripts/dataset_states_to_obs.py` for details on the flags you can use. Make sure you are collect the observations your training loop is expecting. 
```bash
python robomimic/scripts/dataset_states_to_obs.py --dataset <ds-path>
```
The [RoboCasa docs](https://robocasa.ai/docs/use_cases/policy_learning.html) may go into a bit more detail.

### Logging and Viewing Training Results 
#### Experiment Logs
Make sure your `WANDB_ENTITY` and `WANDB_API_KEY` are set in `robomimic/macros_private.py`. 
In your experiment file (e.g. `robomimic/exps/templates/diffusion_policy.json`) edit this:
```bash
"logging": {
    # save terminal outputs under `logs/log.txt` in experiment folder
    "terminal_output_to_txt": true,
    
    # save tensorboard logs under `logs/tb` in experiment folder
    "log_tb": true

    # save wandb logs under `logs/wandb` in experiment folder
    "log_wandb": true
},
```
#### Model Checkpoints
```bash
"save": {
    # enable saving model checkpoints
    "enabled": true,
    
    # controlling frequency of checkpoints
    "every_n_seconds": null,
    "every_n_epochs": 50,
    "epochs": [],
    
    # saving the best checkpoints
    "on_best_validation": false,
    "on_best_rollout_return": false,
    "on_best_rollout_success_rate": true
},
```

#### Evaluating Rollouts and Saving Videos
```bash
"rollout": {
    "enabled": true,              # enable evaluation rollouts
    "n": 50,                      # number of rollouts per evaluation
    "horizon": 400,               # number of timesteps per rollout
    "rate": 50,                   # frequency of evaluation (in epochs)
    "terminate_on_success": true  # terminating rollouts upon task success
}
```
To save videos of the rollpouts, set `render_video` to `True`.
