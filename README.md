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

### Advanced Install

If you already have your own Isaac Lab checkout you want
to reuse, want conda instead of uv, or just want to understand what each
install step does: **[Advanced Install](INSTALL.md)**.

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
- NVIDIA RTX 3090

**`libGLU.so.1: cannot open shared object file`**

If a run fails with an error like:

```
Failed to open /.../isaacsim/extscache/omni.iray.libs-.../bin/iray/libneuray.so: libGLU.so.1: cannot open shared object file: No such file or directory
```

install the missing system library:

```bash
sudo apt-get update && sudo apt-get install -y libglu1-mesa
```

## Acknowledgement

This repository is built upon the support and contributions of the following open-source projects. Special thanks to:

- [IsaacLab](https://github.com/isaac-sim/IsaacLab): The foundation for training and running codes.
- [MuJoCo](https://github.com/google-deepmind/mujoco): Providing powerful simulation functionalities.
- [whole_body_tracking](https://github.com/HybridRobotics/whole_body_tracking): Versatile humanoid control framework for motion tracking.
- [beyondAMP](https://github.com/Renforce-Dynamics/beyondAMP): Referenced for AMP-based motion imitation.
- [mjlab](https://github.com/mujocolab/mjlab): MuJoCo-based training utilities and references.
