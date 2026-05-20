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

# Run with PS5 Controller (Custom Terrain)

## 1) Launch Simulator with Uneven Terrain

```bash
cd /home/dynamics/unitree_rl_mjlab
# Edit simulate/config.yaml so this line is active:
# robot_scene: "src/assets/robots/unitree_a2/xmls/scene_a2_uneven.xml"
./simulate/build/unitree_mujoco --network=enp5s0
```

- Starts MuJoCo with the custom uneven terrain (flat 2x2m center).
- Uses the correct network interface for DDS discovery.

## 2) Launch A2 Controller (PS5 Gamepad)

```bash
cd /home/dynamics/unitree_rl_mjlab/deploy/robots/a2/build
./a2_ctrl --network=enp5s0
```

- Starts the A2 FSM controller.
.

## Gamepad Controls (PS5 → Xbox mapping)
- Cross = A (button 0)
- L2 = LT (axis 2)
- R2 = RT (axis 5)
- D-pad Up = axis 7 (negative)

## Typical FSM Transitions
- Passive → FixStand: Hold L2 + D-pad Up
- FixStand → Velocity: Hold R2 + Cross

> Make sure the PS5 controller is connected before launching the controller. Use `jstest /dev/input/js0` to verify button/axis mapping if needed.

# Gamepad Test

To check if your PS5 controller is detected:

```bash
jstest /dev/input/js0
```
If not installed, run:
```bash
sudo apt install joystick
```

# Controller Launch (No Extra Flags)

Set these in simulate/config.yaml:
```yaml
use_joystick: 1
joystick_type: "xbox"
joystick_device: "/dev/input/js0"
```

Then launch the controller with:
```bash
cd /home/dynamics/unitree_rl_mjlab/deploy/robots/a2/build
./a2_ctrl --network=enp5s0
```

- Do not use --joystick_type or --joystick_device flags (not supported by your binary).



