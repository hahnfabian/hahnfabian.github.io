# Collecting Demos and Training Policies in RoboCasa
(<a href="./index.html">all posts</a>)

1. <a href="#installation">Installation</a>
2. <a href="#setting-up-the-spacemouse-pro">Setting Up the SpaceMouse Pro</a>
    - <a href="#drivers">Drivers</a>
    - <a href="#collecting-demonstrations">Collecting Demonstrations</a>
4. <a href="#policy-learning">Policy Learning</a>
    - <a href="#training">Training</a>
    - <a href="#logging-and-viewing-training-results">Logging and Viewing Training Results</a>

## Installation

To install RoboCasa, follow the comprehensive instructions provided in the [RoboCasa documentation](https://github.com/robocasa/robocasa). **Make sure you are cloning the RoboCasa branches.**
```
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
### Drivers
You will need to install the drivers. Check out the complete documentation at [https://spacenav.sourceforge.net](https://spacenav.sourceforge.net).

```
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
```
sudo spacenavd -v
```
If there is an existing `spacenavd` daemon running you will get an error: `Spacenav daemon already running (pid: 992). Aborting`. Kill this process with `kill -9 992` and start the daemon.

Confirm the install is correct by running the example in `lipsnav/example/cube`. Run `make` and `./cube`. You should be able to move the cube in 3D space with your SpaceMouse.

Finally you need spanv which you can get via pip (?) --> add link to other thing maybe...

### Collecting Demonstrations
You might need to update the `SPACEMOUSE_VENDOR_ID` and `SPACEMOUSE_PRODUCT_ID` in `robosuite/macros_private.py` and `robocasa/macros_private.py` to match your SpaceMouse. You can find these IDs using:

```
lsusb
```
Remember to add `0x` in front if you use the base-16 numbers.

For some SpaceMouse models, you may need to manually activate the logic by modifying the `run` function. Add `or True` to the line as shown below:

``` 
if self.product_id == 50741 or True:
```
This problem is occuring in RoboCasa with the following error message:
```
Exception in thread Thread-1:
Traceback (most recent call last):
  File "/mnt/ssd1/research4/miniconda3/envs/robocasa/lib/python3.9/threading.py", line 980, in _bootstrap_inner
    self.run()
  File "/mnt/ssd1/research4/miniconda3/envs/robocasa/lib/python3.9/threading.py", line 917, in run
    self._target(*self._args, **self._kwargs)
  File "/mnt/ssd1/research4/robosuite/robosuite/devices/spacemouse.py", line 269, in run
    self.roll = convert(d[7], d[8])
IndexError: list index out of range
```

You may need to run as `sudo` for the `hidapi` library. To use the correct Python version (the one from your Conda environment), you will have to replace `python` with `$(which python)`. This ensures that the Python interpreter being used is the one from your Conda environment, rather than the system-wide Python.
```
sudo $(which python) robocasa/scripts/collect_human_demonstrations.py --device spacemouse 
```
To get started with collecting demos, first choose a task from the list provided by [robocasa](https://robocasa.ai/docs/tasks_scenes_assets/atomic_tasks.html). Use the `--environment` flag to indicate which task you want to collect demos for. You can also easily add your own tasks (see further down).

#### Option to disregard demonstrations
Oftentimes a demonstration might not be optimal. I modified `collect_demos.py` so that I can delete a bad demonstration while I'm collecting it. A prerequisite is that you install `keyboard`.
```
pip install keyboard
```
I then added the functionality to the start of the loop in `collect_human_trajectory`.
```
if keyboard.is_pressed('r'):
    print("Disregarding trajectory due to 'R' key press.")
    discard_traj = True
    break
```

## Policy Learning
The easiest option is to use [Robomimic](https://robomimic.github.io/). Once again, make sure to use the RoboCasa branch.
```
git clone https://github.com/ARISE-Initiative/robomimic -b robocasa
cd robomimic
pip install -e .
```

### Training
Each algorithm has a generator script in `scripts/config_gen`. Running this script will give you the commands for your training run(s). To start training you simply need to run the command(s) outputted. To adjust the training change the file in `config_gen` according to the [robomimic docs](https://robomimic.github.io/docs/tutorials/hyperparam_scan.html#step-3-set-hyperparameter-values). 
```
python robomimic/scripts/config_gen/diffusion_gen.py --name <name-to-identify-later>
```

If you want to use your own datasets you will need to convert them from the raw robosuite (RoboCasa datasets are basically robosuite datasets) into the robomimic format. There are two steps.
#### Convert to robomimic

This will convert your dataset in place.
```
python robomimic/scripts/conversion/convert_robosuite.py --dataset <ds-path>
```
#### Extract observations for training

This script will generate a new dataset with the suffix `_im128.hdf5` in the same directory as `--dataset`. There are a bunch of options for what observations are used. Check out `robomimic/scripts/dataset_states_to_obs.py` for details on the flags you can use. Make sure you are collect the observations your training loop is expecting. 
```
python robomimic/scripts/dataset_states_to_obs.py --dataset <ds-path>
```

The [RoboCasa docs](https://robocasa.ai/docs/use_cases/policy_learning.html) may go into a bit more detail.

### Logging and Viewing Training Results 
#### Experiment Logs
Make sure your `WANDB_ENTITY` and `WANDB_API_KEY` are set in `robomimic/macros_private.py`. 
In your experiment file (e.g. `robomimic/exps/templates/diffusion_policy.json`) edit this:
```
"logging": {
    # save terminal outputs under `logs/log.txt` in experiment folder
    "terminal_output_to_txt": true,
    
    # save tensorboard logs under `logs/tb` in experiment folder
    "log_tb": true,

    # save wandb logs under `logs/wandb` in experiment folder
    "log_wandb": true
},
```
#### Model Checkpoints
```
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
```
"rollout": {
    "enabled": true,              # enable evaluation rollouts
    "n": 50,                      # number of rollouts per evaluation
    "horizon": 400,               # number of timesteps per rollout
    "rate": 50,                   # frequency of evaluation (in epochs)
    "terminate_on_success": true  # terminating rollouts upon task success
}
```
To save videos of the rollpouts, set `render_video` to `True`.
