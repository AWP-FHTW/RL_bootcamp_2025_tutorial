# RL Bootcamp 2025

Welcome to **RL Bootcamp 2025**! This repository is a companion resource for the RL Bootcamp, a hands-on, beginner-friendly introduction to **Reinforcement Learning (RL)** using Python. Whether you're an absolute novice or looking to solidify your understanding of RL, this bootcamp is designed to help you gain both foundational concepts and practical experience.

---

## 👥 Who is This For?

This bootcamp is ideal for:
- Students and professionals keen to get started with Reinforcement Learning.
- Learners with basic Python skills and an interest in machine learning or AI.
- Anyone who enjoys hands-on, project-based learning.

No prior experience in RL is required!

---

## 🎯 What Will You Learn?

By completing this bootcamp, you will:
- Understand the core ideas behind Reinforcement Learning, including agents, environments, states, actions, and rewards.
- Get hands-on practice with classic RL algorithms such as Q-learning and Policy Gradients.
- Develop RL agents that can learn through trial and error in simulated environments.
- Explore popular Python libraries for RL (e.g., Gymnasium/OpenAI Gym, NumPy, and others).
- Build intuition for how RL is applied in games, robotics, control, and real-world problems.

---

## 🛠 Prerequisites

- **Python 3.11+** installed on your computer.
- Basic knowledge of Python programming (functions, loops, classes).
- Curiosity about AI, learning, and experimentation!

If you're new to Python or need a refresher, check out the [official Python tutorial](https://docs.python.org/3/tutorial/).

---

## 🏗️ Bootcamp Structure

The course is structured into progressive lessons, each building foundational knowledge and practical skills:
1. **Introduction to Reinforcement Learning**: Concepts and terminology.
2. **Environments & Agents**: How RL tasks are modeled.
3. **Basic RL Algorithms**: Q-learning, SARSA, policy gradients, and more.
4. **Exploring Python RL Libraries**: Getting started with Gymnasium/OpenAI Gym and others.
5. **Hands-on Projects**: Apply your knowledge in coding exercises and mini-projects.

Each lesson includes clear explanations, annotated code examples, readings, and exercises to reinforce your understanding.

---

## 🚀 Getting Started

To begin:
1. Clone or download this repository.
2. Install Python 3.11 (if you don't have it):
   - Windows: Download the Python 3.11 installer from https://www.python.org/downloads/windows/, run it, check "Add python.exe to PATH", and complete setup. Verify with:
     - PowerShell: `py -3.11 --version` (or `python --version` if `py` is unavailable)
   - macOS: Use the official installer from https://www.python.org/downloads/macos/ or Homebrew: `brew install python@3.11`. Verify with `python3.11 --version`.
   - Linux (Ubuntu/Debian): `sudo apt update && sudo apt install -y python3.11 python3.11-venv python3-pip`. Verify with `python3.11 --version`. For other distros, use your package manager (dnf, pacman) to install Python 3.11 along with venv/pip.
3. Set up a virtual environment (recommended):

   ```bash
   python3 -m venv rl-bootcamp-env
   source rl-bootcamp-env/bin/activate  # On Windows: rl-bootcamp-env\Scripts\activate
   ```
4. Install the required packages using the `requirements.txt` file:

   ```bash
   pip install -r requirements.txt
   ```

Check each lesson's notebook or script for additional setup instructions as you progress.

### Windows setup (PowerShell)

If you are on Windows, you can set up everything from PowerShell:

1.  **Install Python 3.12**:
    - Download the latest Python 3.12 installer from the [official website](https://www.python.org/downloads/windows/).
    - When running the installer, **ensure you check the box for "Add python.exe to PATH"**.
    - After installation, open a new PowerShell terminal and verify with `py --version`.

2.  **Create and Activate Virtual Environment**:
    ```powershell
    # Create the virtual environment
    py -m venv rl-bootcamp-env

    # Activate it
    .\rl-bootcamp-env\Scripts\Activate.ps1
    ```
    > **Note:** If you see an error about script execution being disabled, run this command once to allow scripts for your user, then try activating again:
    > `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force`

3.  **Install Dependencies**:
    Once the environment is active (your prompt should start with `(rl-bootcamp-env)`), install the required packages.
    ```powershell
    # Upgrade pip (optional but recommended)
    python -m pip install --upgrade pip

    # Install the project requirements
    pip install -r requirements.txt
    ```

### Single run mode
To run the training code in default configuration defined in `train.yaml` just execute the following code with the virtual environment activated:
```bash
python train.py
```

Trainings with an entirely different configuration are done via:
```bash
python train.py cfg=your_config
```

As mentioned hydra brings the benefit of a hierarchical configuration tool, where every key can be overwritten. E.g. let's run the trainig with a differnt enviornment configuration:
```bash
python train.py env=crippled_ant
```
It is very convinient that hydra stores the configuration in the `logs/runs/../<run_dir>` directory along with a list defining the overwritten keys.
 

It is a good praxis to take advantage of the hierarchy by using a well definied default configuration and overwrite only neccessary parts in an experiment file:
```bash
python train.py experiment=your_custom_experiment
```

Let's for example define change of the enviorment configuration entirly and modify some parameters like the number of training evnironments used and a enviroment parameter which is passed to the constructor of the gym enviorment. Be careful not to forget ```# @package _global_``` right before the defaults list, as this tells hydra to merge configurations in the global configuration space.

```yaml
# @package _global_
defaults:
  - override /env: crippled_ant

train_env:
  n_envs: 6           # increase number of training environments
  env_kwargs:
    injury: medium    # disable two instead of one leg

# define a proper task name making it easier to link results with configurations
task_name: "train_${env.id}_${env.train_env.env_kwargs.injury}"
```

### Multirun mode (-m)
One of the major advantages of hydra is that it provides multirun support.
Consider e.g. the follwing case where we want to run the training with three differnt configurations for the learning rate:
```bash
python train.py -m agent.learning_rate=1e-4,5e-4,1e-3
```
Hydra creates now three run directories in `logs/multiruns/...` where the results and configurations stored similar to the single run case.

Per default this jobs are executed sequentally which is not the workflow suited to train reinforcement learning agents. Luckily, this can be very easily fixed since hydra offers several plugins for job launching. Consider e.g. the following configuration for hyperparmeter tuning:

```yaml
# @package _global_
defaults:
  - override /hydra/launcher: ray
  - override /hydra/sweeper: optuna

hydra:
  mode: "MULTIRUN"
  launcher:
    ray:
      remote:
        num_cpus: 4
      init:
        local_mode: false

  sweeper:
    _target_: hydra_plugins.hydra_optuna_sweeper.optuna_sweeper.OptunaSweeper
    n_trials: 20
    n_jobs: 4
    direction: minimize

    sampler:
      _target_: optuna.samplers.TPESampler
      seed: ${seed}
      n_startup_trials: 10
```

```override /hydra/launcher: ray``` essentially tells hydra to use the ray plugin for job launching which is defined below. In the present case we use 4 CPUs. In addition to launcher plugin we also take advantage of the optuna plugin via ```override /hydra/sweeper: optuna``` which gives as access to more elaborated hyperparameter sampling. In the present case we use the TPESampler which comes with a bit of intelligence instead of brute force grid sampling.

The next step now is to define the parameters we want to optimize for which is again best done via an experiment configuration. E.g. let's create a ```hp_ant_baseline.yaml``` experiment file, essentially loading the the relvant plugins via ```override /hparams_search: optuna``` and definig the parameter space for the learn rate and the clip range for PPO which are the parameters in this example we want to optimize for.


```yaml
# @package _global_
defaults:
  - override /hparams_search: optuna

task_name: "hparams_search_PPO@ANT"

hydra:
  sweeper:
    params:
      agent.clip_range: interval(0.05, 0.3)
      agent.learning_rate: interval(0.0001, 0.01)

learner.total_timesteps: 1000000

# Since we optimize for minimum training time we need early stopping defined
callbacks:
  eval_callback:
    callback_on_new_best:
      _target_: stable_baselines3.common.callbacks.StopTrainingOnRewardThreshold
      reward_threshold: 1000
      verbose: 1
```

Again to run the hyperparameter search we just need to run hydra in multrun mode with configuration we defined above.

```bash
python train.py -m experiment=hp_ant_baseline
```



### Repository Structure

    RL_bootcamp_2025_tutorial/
    ├── config/                 # Hydra configuration files
    │   ├── agent/              # Agent-specific settings
    │   ├── callbacks/          # Callbacks during training/evaluation
    │   ├── env/                # Environment definitions and parameters
    │   ├── experiment/         # Experiment configuration files
    │   ├── hparams_search/     # Hyperparameter search configs
    │   ├── learner/            # Learning wrapper configs
    │   ├── policy/             # Policy architecture and parameters
    │   ├── hparams_search/     # Hyperparameter search configs
    │   └── train.yaml          # Main training configuration
    ├── src/                    # Core source code
    │   ├── envs/               # Environment source code
    │   ├── models/             # Neural net definitons for feature extractors
    │   ├── utils/              # Helpers for instantiation and postprocessing
    │   └── wrappers/           # Code wrappers 
    ├── inference.py            # Inference script evaluating policy snapshots
    ├── train.py                # Main training entry point
    ├── vanilla_train.py        # A simple scipt to train without hydra (aimed for visualization, not recommended to use)
    ├── requirements.txt        # Python dependencies
    ├── README.md               # This file
    └── LICENSE                 # License information









## 🤝 Contributing and Questions

We welcome feedback and questions! Please use the Issues tab or reach out as directed on the course website.

Happy learning and experimenting in RL Bootcamp 2025!