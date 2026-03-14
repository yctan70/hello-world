  
 # topstar_mujoco

MuJoCo simulator for the Topstar H2 robot. Derived from [unitree_mujoco](https://github.com/unitreerobotics/unitree_mujoco) but adapted to use CycloneDDS directly — no `unitree_sdk2` dependency.

## Directory structure

| Path | Description |
|---|---|
| `simulate/` | C++ MuJoCo simulator (recommended) |
| `simulate_python/` | Python MuJoCo simulator |
| `topstar_robots/h2/` | H2 robot `MJCF` model and meshes |
| `terrain_tool/` | Tool for generating terrain scenes |
| `example/` | Example control programs |

## DDS messages

The simulator bridges MuJoCo state to/from CycloneDDS using the `topstar_hg`
IDL types (defined in `simulate/src/topstar_hg.h`):

| Topic | Type | Direction |
|---|---|---|
| `rt/lowstate` | `topstar_hg_msg_dds__LowState_` | simulator → controller |
| `rt/lowcmd` | `topstar_hg_msg_dds__LowCmd_` | controller → simulator |

The H2 robot has 30 DOF. The IDL allocates 35 motor slots; the mapping is:

| Slots | Joints |
|---|---|
| 0 – 5 | Left leg |
| 6 – 11 | Right leg |
| 12 – 13 | Waist |
| 14 – 15 | Head |
| 16 – 22 | Left arm |
| 23 – 29 | Right arm |

---

# Installation

## Dependencies

```bash
sudo apt install libyaml-cpp-dev libspdlog-dev libboost-all-dev libglfw3-dev
```
CycloneDDS must be installed at `/usr/local` (headers in `/usr/local/include`, library at `/usr/local/lib/libddsc.so`).  The CMake build detects it automatically; no `pkg-config` entry is required.

MuJoCo 3.x must be installed. Download a [release](https://github.com/google-deepmind/mujoco/releases), extract to `~/.mujoco/`, then symlink it into the simulator source tree:
```bash
ln -s ~/.mujoco/mujoco-3.x.x topstar_mujoco/simulate/mujoco
```

---

# C++ simulator

## Build

```bash
cd topstar_mujoco/simulate
mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
```

The binary is produced at `simulate/build/topstar_mujoco`.

`compile_commands.json` is generated automatically and symlinked to the
workspace root for IDE / clangd support.

## Configure

Edit `simulate/config.yaml`:

```yaml
robot: "h2"          # Robot name — selects topstar_robots/h2/
robot_scene: "scene.xml"

domain_id: 0         # CycloneDDS domain id
interface: "eno1"    # Network interface (use "lo" for loopback)

use_joystick: 0      # 1 to forward a USB gamepad as wireless_remote
joystick_type: "xbox"
joystick_device: "/dev/input/js0"

print_scene_information: 1   # Print joint/sensor list on startup
enable_elastic_band: 1       # Virtual suspension band (see below)
```

## Run

```bash
cd simulate/build
./topstar_mujoco
```

Override robot or scene on the command line:

```bash
./topstar_mujoco -r h2 -s scene.xml
./topstar_mujoco -n lo   # loopback interface
```

---

# Python simulator

## Dependencies

```bash
pip3 install mujoco pygame
```

The Python bridge uses `ctypes` to call `libddsc.so` and
`libtopstar_hg.so` directly — no `unitree_sdk2py` is required.

Build `libtopstar_hg.so` once before first use:

```bash
gcc -shared -fPIC -O2 -I/usr/local/include \
    simulate/src/topstar_hg.c \
    -o simulate_python/libtopstar_hg.so -lddsc
```

## Configure

Edit `simulate_python/config.py`:

```python
ROBOT = "h2"
ROBOT_SCENE = "../topstar_robots/" + ROBOT + "/scene.xml"

DOMAIN_ID = 0
INTERFACE = "eno1"   # use "lo" for loopback

USE_JOYSTICK = 0
JOYSTICK_TYPE = "xbox"
JOYSTICK_DEVICE = 0

PRINT_SCENE_INFORMATION = True
ENABLE_ELASTIC_BAND = True

SIMULATE_DT = 0.005   # must exceed viewer.sync() runtime
VIEWER_DT   = 0.02    # 50 fps
```

## Run

```bash
cd simulate_python
python3 topstar_mujoco.py
```

---

# Elastic band

The virtual elastic band suspends the robot in mid-air for safe initialisation.  Enable it with `enable_elastic_band: 1` / `ENABLE_ELASTIC_BAND = True`.

The band is attached to `head_pitch_Link` and anchored at a point 3 m above the origin.  The default rest length is set so the robot hangs with its feet **0.3 m above the ground**.

| Key | Action |
|---|---|
| `9` | Toggle band on / off |
| `7` | Lower robot (shorten rest length −0.1 m) |
| `8` | Raise robot (lengthen rest length +0.1 m) |

---

# Terrain generation

The `terrain_tool/` directory contains a parametric terrain generator for stairs, rough ground, and height maps.  See `terrain_tool/README.md` for usage.

---

# Joystick layout

Set `use_joystick: 1` to forward a physical gamepad as the `wireless_remote` bytes in `LowState`.  Xbox and Switch layouts are supported.  To add a custom layout, edit `simulate/src/physics_joystick.h` (C++) or `simulate_python/topstar_bridge.py` (Python) and map axis/button IDs — use `jstest /dev/input/jsN` to discover them.



