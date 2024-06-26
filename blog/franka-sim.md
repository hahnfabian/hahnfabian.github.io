# Simulation a Franka Panda 
Simulating the Franka Emika Panda robot  with <a href="https://github.com/google-deepmind/mujoco">MuJoCo</a> provides an invaluable platform for developers to experiment, test, and refine their algorithms in a risk-free environment. This technical guide will walk you through the steps and best practices for setting up and utilizing a Franka robot simulation.

%%% image/video here %%%
### Chapters
1. <a href="#installation">Installation</a>
2. <a href="#basics">Basics</a>
3. <a href="#robocasa">Robocasa</a>
 
## Installation
#### Step 1: Install Conda
If you haven't already installed Conda, you can download and install Miniconda or Anaconda from the following links:
- <a href="https://docs.anaconda.com/miniconda/miniconda-install">Miniconda</a>
- <a href="https://www.anaconda.com/download">Anaconda</a>

#### Step 2: Create a New Conda Environment
Open your terminal or command prompt and create a new Conda environment. Here, we'll create an environment named ```franka_sim```: 
``` 
conda create --name franka_sim python=3.8
```
Activate the new environment:
```
conda activate franka_sim
```

#### Step 3: Install MuJoCo
Visit the <a href="https://github.com/google-deepmind/mujoco/releases">MuJoCo Website</a> and download the appropriate version for your operating system.

Extract the downloaded directory into ```~/.mujoco/mujoco<version> ```
Finally run:
```
pip install mujoco
```

#### Step 4: Verify Installation 
Go to ```~/.mujoco/mujoco<version>/bin``` and run ```./simulate```.
