# A2 Training Commands Reference

## Basic Commands

### Train on Flat Terrain (Simple)
```bash
python scripts/train.py Unitree-A2-Flat --env.scene.num-envs=4096
```

### Train on Rough Terrain (Recommended for Real Robot)
```bash
python scripts/train.py Unitree-A2-Rough --env.scene.num-envs=4096
```

### Multi-GPU Training
```bash
python scripts/train.py Unitree-A2-Rough \
  --gpu-ids 0 1 \
  --env.scene.num-envs=4096
```

---

## Advanced: Full Command with Parameters

```bash
python scripts/train.py Unitree-A2-Rough \
  --env.scene.num-envs=4096 \
  --env.commands.twist.ranges.lin_vel_x=[-1.5,2.5] \
  --env.commands.twist.ranges.lin_vel_y=[-1.5,1.5] \
  --env.commands.twist.ranges.ang_vel_z=[-2.0,2.0] \
  --env.events.foot_friction.params.ranges=[0.2,1.5] \
  --env.events.encoder_bias.params.bias_range=[-0.02,0.02] \
  --agent.max_iterations=20000 \
  --agent.algorithm.learning_rate=5e-4 \
  --agent.algorithm.entropy_coef=0.02 \
  --agent.seed=42
```

---

## Parameter Explanations

### Environment Parameters

| Parameter | Default | Range | Explanation |
|-----------|---------|-------|-------------|
| `--env.scene.num-envs` | 4096 | 512-8192 | Number of parallel environments (more = faster but more VRAM) |
| `--env.sim.dt` | 0.005 | 0.001-0.01 | Simulation timestep (smaller = more accurate) |
| `--env.seed` | 0 | any | Random seed for reproducibility |

### Command Ranges (Velocities Robot Learns)

| Parameter | Default | Explanation |
|-----------|---------|-------------|
| `--env.commands.twist.ranges.lin_vel_x` | [-1.0, 2.0] | Forward velocity: min to max m/s |
| `--env.commands.twist.ranges.lin_vel_y` | [-1.0, 1.0] | Lateral velocity: min to max m/s |
| `--env.commands.twist.ranges.ang_vel_z` | [-1.0, 1.0] | Rotation velocity: min to max rad/s |

**Tips:**
- Smaller ranges = easier training, less robust
- Larger ranges = harder training, better generalization
- For rough terrain, use: `[-1.5, 2.5]`, `[-1.5, 1.5]`, `[-2.0, 2.0]`

### Domain Randomization (Sim-to-Real Gap)

| Parameter | Default | Explanation |
|-----------|---------|-------------|
| `--env.events.foot_friction.params.ranges` | [0.3, 1.2] | Foot friction randomization |
| `--env.events.encoder_bias.params.bias_range` | [-0.015, 0.015] | Joint sensor noise |
| `--env.events.base_com.params.ranges` | varies | COM offset randomization |

**Tips:**
- Higher ranges = better real robot transfer but slower training
- For robust policy: `[0.2, 1.5]` for friction

### RL Algorithm Parameters

| Parameter | Default | Explanation |
|-----------|---------|-------------|
| `--agent.max_iterations` | 10001 | Total training iterations |
| `--agent.algorithm.learning_rate` | 1e-3 | Learning rate (higher = faster but less stable) |
| `--agent.algorithm.entropy_coef` | 0.01 | Exploration bonus (higher = more random) |
| `--agent.algorithm.num_learning_epochs` | 5 | Optimization passes per batch |
| `--agent.algorithm.clip_param` | 0.2 | Policy update clipping |
| `--agent.num_steps_per_env` | 24 | Steps collected before batch update |

**Tips:**
- For rough terrain: increase `max_iterations` to 15000-20000
- Stable learning: use learning_rate = 5e-4
- More exploration: entropy_coef = 0.02-0.03

### Network Architecture

| Parameter | Default | Explanation |
|-----------|---------|-------------|
| `--agent.actor.hidden_dims` | [512, 256, 128] | Network layer sizes |
| `--agent.actor.activation` | elu | Activation function |

---

## Recommended Configurations

### Quick Test (Fast Convergence, Less Robust)
```bash
python scripts/train.py Unitree-A2-Flat \
  --env.scene.num-envs=2048 \
  --agent.max_iterations=5000
```

### Production (Rough Terrain, Robust, Slower)
```bash
python scripts/train.py Unitree-A2-Rough \
  --env.scene.num-envs=4096 \
  --env.commands.twist.ranges.lin_vel_x=[-1.5,2.5] \
  --env.commands.twist.ranges.lin_vel_y=[-1.5,1.5] \
  --env.events.foot_friction.params.ranges=[0.2,1.5] \
  --agent.max_iterations=20000 \
  --agent.algorithm.learning_rate=5e-4 \
  --agent.algorithm.entropy_coef=0.02
```

### Maximum Robustness (Real Hardware)
```bash
python scripts/train.py Unitree-A2-Rough \
  --env.scene.num-envs=4096 \
  --env.commands.twist.ranges.lin_vel_x=[-2.0,3.0] \
  --env.commands.twist.ranges.lin_vel_y=[-2.0,2.0] \
  --env.commands.twist.ranges.ang_vel_z=[-2.5,2.5] \
  --env.events.foot_friction.params.ranges=[0.1,2.0] \
  --env.events.encoder_bias.params.bias_range=[-0.03,0.03] \
  --agent.max_iterations=30000 \
  --agent.algorithm.learning_rate=3e-4 \
  --agent.algorithm.entropy_coef=0.03 \
  --agent.algorithm.num_learning_epochs=8
```

---

## Resume Training

```bash
python scripts/train.py Unitree-A2-Rough \
  --agent.resume=true \
  --agent.load_run=.* \
  --agent.load_checkpoint=model_*.pt
```

---

## Playback

### Test Flat-Trained Model
```bash
python scripts/play.py Unitree-A2-Flat \
  --checkpoint_file ~/unitree_rl_mjlab/logs/rsl_rl/a2_velocity/YYYY-MM-DD_HH-MM-SS/model_5000.pt \
  --viewer viser
```

### Test Rough-Trained Model
```bash
python scripts/play.py Unitree-A2-Rough \
  --checkpoint_file ~/unitree_rl_mjlab/logs/rsl_rl/a2_velocity/YYYY-MM-DD_HH-MM-SS/model_10000.pt \
  --viewer viser
```

### Transfer Test (Flat Model on Rough Terrain)
```bash
python scripts/play.py Unitree-A2-Transfer-Rough \
  --checkpoint_file ~/unitree_rl_mjlab/logs/rsl_rl/a2_velocity/2026-03-26_10-32-41/model_500.pt \
  --viewer viser
```

---

## GPU Selection

### Single GPU (GPU 0)
```bash
CUDA_VISIBLE_DEVICES=0 python scripts/train.py Unitree-A2-Rough --env.scene.num-envs=4096
```

### Multiple GPUs
```bash
python scripts/train.py Unitree-A2-Rough --gpu-ids 0 1 2 --env.scene.num-envs=4096
```

### CPU Only
```bash
CUDA_VISIBLE_DEVICES="" python scripts/train.py Unitree-A2-Flat --env.scene.num-envs=512
```

---

## Output

Logs saved to: `logs/rsl_rl/a2_velocity/<date>_<time>/`
- `model_*.pt` — Checkpoints every 100 iterations
- `policy.onnx` — Exported policy for deployment
- `params/env.yaml` — Environment configuration
- `params/agent.yaml` — RL configuration
