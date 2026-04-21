# Roboracer Workspace (Coadaptive LLM Fork)

This is a custom fork of the UT AV Roboracer workspace, specifically modified to support our Coadaptive LLM Reinforcement Learning pipeline using Qwen2.5. It contains the standard ROS2 packages to control the 1/10th scale autonomous racecar, alongside our custom reward shaping and telemetry tracking scripts.

For general setup and standard usage, refer to the [UT AV Getting Started](https://ut-av.pages.dev/getting_started/) docs.

## System Requirements

The codebase automatically detects your platform and switches between two modes:

1. **Simulation Mode:** Active when running on `amd64` or MacOS `arm64`. (This is where our custom LLM training pipeline runs).
2. **Hardware Mode:** Active when an NVIDIA Jetson Orin device is detected. 
> *Note: For hardware mode firmware flashing and setup, see the [ot_orin_ros2](https://github.com/FRI-Self-Driving/ot_orin_ros2.git) repository.*

**Crucial Path Requirement:** You absolutely must name this workspace `roboracer_ws` and clone it directly into your home directory (`~/roboracer_ws`). The codebase relies on this exact path to find the configuration files, and it will break if you put it anywhere else.

## Quickstart: Coadaptive Training Pipeline

Make sure you have [Docker](https://ut-av.pages.dev/tools/docker/) installed according to the main documentation before starting. 

Here is the exact pipeline to get the simulator running with our custom Qwen reward loop:

```bash
# 1. Clone the custom fork exactly into the home directory
cd ~
git clone [https://github.com/Spacewalker215/roboracer_ws.git](https://github.com/Spacewalker215/roboracer_ws.git)
cd roboracer_ws

# 2. Build and enter the Docker container
./container build
./container shell

# 3. Spin up the virtual display for the Unity simulator
vnc
export DISPLAY=:9

# 4. Install our custom dependencies (OpenAI client for the vLLM server)
cd simulator
uv pip install openai --python ~/.venv-simulator

# 5. Kick off the PPO training run with the LLM in the loop
# Note: We assign QWEN_PORT to avoid overlapping with other users on the lab machine
OPENBLAS_NUM_THREADS=2 MKL_NUM_THREADS=2 OMP_NUM_THREADS=2 VECLIB_MAXIMUM_THREADS=2 NUMEXPR_NUM_THREADS=2 \
QWEN_PORT=8000 \
uv run python src/algorithms/ppo_pufferlib.py \
  --random-spawn --random-spawn-max-cte-offset 0.8 --random-spawn-max-rotation-offset 10.0 \
  --num-envs 6 --start-port 9301 --min-throttle 0.2 \
  --total-timesteps 2000000 \
  --env-name donkey-roboracingleague-track-v0 \
  --log-dir ./output/tb_runC_full \
  --model-dir ./output/models_runC_full \
  --seed 1
  ```
