# A2 Transfer-Rough: Working Model Path

The deployed controller model was copied from:

- `logs/rsl_rl/a2_velocity/2026-03-26_10-32-41/policy.onnx`

to:

- `deploy/robots/a2/config/policy/velocity/v0/exported/policy.onnx`

This destination is inside this repo, so it can be tracked in git like any other file.

# Launch Commands

## 1) Launch Simulator

Command:

```bash
cd /home/dynamics/unitree_rl_mjlab
./simulate/build/unitree_mujoco --network=enp5s0
```

Explanation:

- Starts MuJoCo simulation and Unitree SDK bridge.
- Uses `enp5s0` so DDS discovery works on this machine.

## 2) Launch A2 Controller

Command:

```bash
cd /home/dynamics/unitree_rl_mjlab/deploy/robots/a2/build
./a2_ctrl --network=enp5s0
```

Explanation:

- Starts the A2 FSM controller.
- Loads policy from `deploy/robots/a2/config/policy/velocity/v0/exported/policy.onnx`.
- After startup, use gamepad transitions to enter control mode.

# Optional Validation Command (Play)

Command:

```bash
python /home/dynamics/unitree_rl_mjlab/scripts/play.py Unitree-A2-Transfer-Rough --checkpoint_file /home/dynamics/unitree_rl_mjlab/logs/rsl_rl/a2_velocity/2026-03-26_10-32-41/model_10000.pt --viewer viser
```

Explanation:

- Verifies the checkpoint behavior in play mode.

Optional Validation Screenshot:

![A2 transfer-rough validation result](doc/gif/a2_transfer_rough_validation.png)



