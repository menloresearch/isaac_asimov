# Isaac Asimov

Standalone Isaac Lab extension for training Asimov-1 locomotion policies with
PPO and adversarial motion priors (AMP).

## Quick Install

For a brand-new machine with no existing Isaac Lab setup, run the install
script from the project root. It sets up a uv-managed environment, pulls the
pinned Isaac Lab and `asimov-1` submodules, and installs everything needed
to train and play. Prerequisites: Ubuntu 22.04+ (x86_64), a compatible
NVIDIA driver, [uv](https://docs.astral.sh/uv/) installed, and `sudo` access
(used to install `cmake`/`build-essential`).

```bash
git clone https://github.com/menloresearch/isaac_asimov.git
cd isaac_asimov
./quick_install.sh
```

## Advanced Install

Use this instead of `quick_install.sh` if you already have your own Isaac
Lab checkout you want to reuse, want conda instead of uv, or just want to
understand what each install step does. Pick one of the two approaches
below; both end with the same "Install this extension" step.

**Contents**
- [Using your own Isaac Lab checkout](#using-your-own-isaac-lab-checkout)
- [Install with uv](#install-with-uv)
- [Install with conda and pip](#install-with-conda-and-pip)
- [Robot model](#robot-model)
- [Install this extension](#install-this-extension)

### Using your own Isaac Lab checkout

If you already have Isaac Lab cloned somewhere, skip the
`git submodule update --init third_party/IsaacLab` step in whichever
approach you follow below, and substitute your own path wherever
`third_party/IsaacLab` appears (for example,
`(cd /path/to/your/IsaacLab && ./isaaclab.sh --install rsl_rl)`). Everything
else — the environment setup, the robot model, and installing this
extension — stays the same. This project pins RSL-RL 5.0.1 and expects
Isaac Lab `main` (currently reporting 2.3.2, with RSL-RL 5 support); other
versions may not be compatible.

### Install with uv

These steps target Ubuntu 22.04+ (x86_64), with
[uv](https://docs.astral.sh/uv/) and a compatible NVIDIA driver installed.

1. Install build tools needed to compile some Isaac Lab dependencies:

   ```bash
   sudo apt-get install -y cmake build-essential
   ```

2. Fetch Isaac Lab, pinned to a known-working commit (skip this if using
   your own checkout — see above):

   ```bash
   git submodule update --init third_party/IsaacLab
   ```

3. Create a Python 3.11 virtual environment and activate it. `--seed`
   installs `pip` into the venv, since Isaac Lab's own installer shells out
   to it:

   ```bash
   uv venv --seed --python 3.11
   source .venv/bin/activate
   ```

4. Install Isaac Sim and CUDA-enabled PyTorch into the active environment:

   ```bash
   uv pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com
   uv pip install torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
   ```

5. Install Isaac Lab's RSL-RL extras. This installs `isaaclab`,
   `isaaclab_rl`, and `isaaclab_tasks` in editable mode from the checkout,
   plus `rsl-rl-lib`:

   ```bash
   (cd third_party/IsaacLab && ./isaaclab.sh --install rsl_rl)
   ```

See the
[official installation guide](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html)
for prerequisites and troubleshooting.

### Install with conda and pip

These steps target Ubuntu 22.04+ (x86_64), with Conda and a compatible
NVIDIA driver installed.

1. Install build tools needed to compile some Isaac Lab dependencies:

   ```bash
   sudo apt-get install -y cmake build-essential
   ```

2. Fetch Isaac Lab, pinned to a known-working commit (skip this if using
   your own checkout — see above):

   ```bash
   git submodule update --init third_party/IsaacLab
   ```

3. Create a Python 3.11 conda environment and activate it:

   ```bash
   conda create -n env_isaaclab python=3.11 -y
   conda activate env_isaaclab
   python -m pip install --upgrade pip
   ```

4. Install Isaac Sim and CUDA-enabled PyTorch into the active environment:

   ```bash
   python -m pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com
   python -m pip install torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
   ```

5. Install Isaac Lab's RSL-RL extras. This installs `isaaclab`,
   `isaaclab_rl`, and `isaaclab_tasks` in editable mode from the checkout,
   plus `rsl-rl-lib`:

   ```bash
   (cd third_party/IsaacLab && ./isaaclab.sh --install rsl_rl)
   ```

Keep `env_isaaclab` active for the remaining steps. See the
[official installation guide](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html)
for prerequisites and troubleshooting.

### Robot model

The `asimov-1` submodule provides the robot's URDF and meshes at
`third_party/asimov-1/sim-model`; the robot configuration uses this location
by default, so no path changes are needed. Its repository is large (mostly
unrelated CAD/fabrication files), so this fetches only the `sim-model`
subtree via sparse-checkout:

```bash
git submodule update --init --filter=blob:none third_party/asimov-1
git -C third_party/asimov-1 sparse-checkout set sim-model
```

### Install this extension

With the environment from either approach above still active, install this
extension in editable mode:

```bash
./isaac_asimov.sh --install
```

## Train
**Quick Test**
create a small job to see if the full training code is working end2end. These settings should work for most gpus and finished relatively quickly.

```bash
# AMP (recommended)
./isaac_asimov.sh --train \
    --task Asimov1-Velocity-AMP-v0 --num_envs 128 --headless --max_iterations 100
```

**Single GPU**

```bash
# AMP (recommended)
./isaac_asimov.sh --train \
    --task Asimov1-Velocity-AMP-v0 --num_envs 4096 --headless

# Plain PPO baseline
./isaac_asimov.sh --train \
    --task Asimov1-Velocity-v0 --num_envs 4096 --headless
```

Useful flags: `--max_iterations <n>`, `--seed <n>`, `--video` (record rollout
clips during training).

Note: We use 4096 environments to train our baseline locomotion policy using A6000 or pro 6000. You may need to reduce the number of environments if you are training on a gpu with less memory.

**Multi-GPU / distributed** training via `--distributed` (two GPUs, 4096
environments per GPU):

```bash
python -m torch.distributed.run --standalone --nnodes=1 --nproc_per_node=2 \
    scripts/rsl_rl/train.py \
    --task Asimov1-Velocity-AMP-v0 --num_envs 4096 --headless --distributed
```

## Play / Evaluate

Load the latest checkpoint and visualize the trained policy:

```bash
./isaac_asimov.sh --play \
    --task Asimov1-Velocity-AMP-Play-v0 --num_envs 32
```

Use `--checkpoint <path>` to select a specific checkpoint (`--target` is an
alias), or `--onnx-output <path>` for an extra ONNX export.

Checkpoints and logs are written to `logs/rsl_rl/<experiment_name>/<run>/`.

## Troubleshooting
The training code has been tested on the following GPUs:
- NVIDIA RTX A6000
- NVIDIA RTX PRO 6000
- NVIDIA RTX 4090

## Acknowledgement

This repository is built upon the support and contributions of the following open-source projects. Special thanks to:

- [IsaacLab](https://github.com/isaac-sim/IsaacLab): The foundation for training and running codes.
- [MuJoCo](https://github.com/google-deepmind/mujoco): Providing powerful simulation functionalities.
- [whole_body_tracking](https://github.com/HybridRobotics/whole_body_tracking): Versatile humanoid control framework for motion tracking.
- [beyondAMP](https://github.com/Renforce-Dynamics/beyondAMP): Referenced for AMP-based motion imitation.
- [mjlab](https://github.com/mujocolab/mjlab): MuJoCo-based training utilities and references.
