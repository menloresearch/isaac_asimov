# Isaac Asimov

Standalone Isaac Lab extension for training Asimov-1 locomotion policies with
PPO and adversarial motion priors (AMP).

## Install Isaac Sim and Isaac Lab

These steps target Ubuntu 22.04+ (x86_64), with [uv](https://docs.astral.sh/uv/)
and a compatible NVIDIA driver installed. Use Python 3.11, Isaac Sim 5.1.0,
and the pinned Isaac Lab submodule (currently reporting 2.3.2, with RSL-RL 5
support). This project pins RSL-RL 5.0.1.

Isaac Lab and the `asimov-1` robot model are vendored as git submodules
under `third_party/`, pinned to known-working commits. Initialize them:

```bash
git submodule update --init third_party/IsaacLab
git submodule update --init --filter=blob:none third_party/asimov-1
git -C third_party/asimov-1 sparse-checkout set sim-model
```

Create the environment and install Isaac Sim and CUDA-enabled PyTorch:

```bash
uv venv --seed --python 3.11
source .venv/bin/activate
uv pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com
uv pip install torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
```

Install Isaac Lab's RSL-RL extras into the active environment:

```bash
sudo apt-get install -y cmake build-essential
(cd third_party/IsaacLab && ./isaaclab.sh --install rsl_rl)
```

Keep `.venv` active for the remaining steps. See the
[official installation guide](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html)
for prerequisites and troubleshooting.

## Robot model

The `asimov-1` submodule (initialized above) provides the URDF and meshes at
`third_party/asimov-1/sim-model`, sparse-checked-out to avoid pulling the
rest of that repo (CAD/fabrication files). The robot configuration uses this
location by default; no path changes are needed.

## Quick start

### 1. Set up

Activate the environment in which Isaac Lab is installed, then install this
extension in editable mode:

```bash
source .venv/bin/activate
./isaac_asimov.sh --install
```

### 2. Train

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

**Multi-GPU / distributed** training via `--distributed` (two GPUs, 4096
environments per GPU):

```bash
python -m torch.distributed.run --standalone --nnodes=1 --nproc_per_node=2 \
    scripts/rsl_rl/train.py \
    --task Asimov1-Velocity-AMP-v0 --num_envs 4096 --headless --distributed
```

### 3. Play / Evaluate

Load the latest checkpoint and visualize the trained policy:

```bash
./isaac_asimov.sh --play \
    --task Asimov1-Velocity-AMP-Play-v0 --num_envs 32
```

Use `--checkpoint <path>` to select a specific checkpoint (`--target` is an
alias), or `--onnx-output <path>` for an extra ONNX export.

Checkpoints and logs are written to `logs/rsl_rl/<experiment_name>/<run>/`.

## Acknowledgement

This repository is built upon the support and contributions of the following open-source projects. Special thanks to:

- [IsaacLab](https://github.com/isaac-sim/IsaacLab): The foundation for training and running codes.
- [MuJoCo](https://github.com/google-deepmind/mujoco): Providing powerful simulation functionalities.
- [whole_body_tracking](https://github.com/HybridRobotics/whole_body_tracking): Versatile humanoid control framework for motion tracking.
- [beyondAMP](https://github.com/Renforce-Dynamics/beyondAMP): Referenced for AMP-based motion imitation.
- [mjlab](https://github.com/mujocolab/mjlab): MuJoCo-based training utilities and references.
