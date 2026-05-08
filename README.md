## OpenRouter camera descriptions

Build and source the ROS 2 workspace, then export your OpenRouter key:

```bash
colcon build --packages-select qbot_platform --symlink-install
source install/setup.bash
export OPENROUTER_API_KEY="sk-or-v1-..."
```

Check the active camera topic first:

```bash
ros2 topic list
```

Start the QBot RGBD camera node and keep this terminal open:

```bash
ros2 run qbot_platform rgbd --ros-args -r __node:=rgbd_camera
```

For this package's RGBD node topic shown by `ros2 topic list`:

```bash
ros2 run qbot_platform openrouter_camera_stream.py \
  --topic /camera/color_image \
  --interval 10
```

For the default RealSense ROS 2 wrapper topic:

```bash
ros2 run qbot_platform openrouter_camera_stream.py \
  --topic /camera/camera/color/image_raw \
  --qos sensor_data \
  --interval 10
```

The node prints what `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free` sees and also publishes
the text on:

```bash
ros2 topic echo /openrouter/camera_description
```

If raw images are advertised but not flowing, use the compressed stream:

```bash
ros2 run qbot_platform openrouter_camera_stream.py \
  --topic /camera/color_image/compressed \
  --input-type compressed \
  --interval 10
```

To verify the script is sending fresh images, run without OpenRouter and compare
the printed `stamp`, `input_hash`, and `sent_hash` values:

```bash
ros2 run qbot_platform openrouter_camera_stream.py \
  --topic /camera/color_image/compressed \
  --input-type compressed \
  --interval 2 \
  --diagnose-only \
  --save-debug-frames debug_frames/openrouter-camera-debug
```

## Vision-gated forward motion

Export your OpenRouter key, then launch the QBot driver stack, camera,
odometry, and vision-forward agent together:

```bash
source install/setup.bash
export OPENROUTER_API_KEY="sk-or-v1-..."

ros2 launch qbot_platform qbot_platform_vision_forward_launch.py attempts:=3
```

The launch defaults cap OpenRouter reasoning tokens and exclude returned
reasoning text so the model has enough budget to return the required JSON.

For a no-motion test that still asks the model for a clear/blocked verdict:

```bash
ros2 launch qbot_platform qbot_platform_vision_forward_launch.py \
  attempts:=3 \
  dry_run:=true
```

Watch the agent events:

```bash
ros2 topic echo /vision_forward_agent/status
```
