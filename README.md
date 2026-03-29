
# `ros-mcp-server/README.md`

````markdown
# ros-mcp-server

## Overview
`ros-mcp-server` is an MCP-based interface server for a ROS 2 machining / polishing pipeline.  
This repository provides a natural-language interaction layer between Gemini CLI and the local robot-control / data-processing environment.

Its main roles are:
- natural-language command interpretation
- MCP-compatible server execution
- optional microphone/STT-based command input
- optional camera/image input handling
- communication with the ROS-side execution workspace

This repository does **not** by itself perform the full ROS-side machining workflow.  
Actual trajectory execution, raw data collection, filtering, and analysis are handled in the companion ROS 2 workspace repository.

---

## Repository Structure

```text
ros-mcp-server/
├── .github/                # GitHub workflows
├── camera/                 # image input / received images
├── prompts/                # prompt templates
├── stt/                    # speech-to-text related scripts
├── utils/                  # helper utilities
├── server.py               # main MCP server entry point
├── pyproject.toml          # Python project configuration
├── uv.lock                 # dependency lock file
├── GEMINI.md               # Gemini-related notes / environment guidance
└── README.md
````

---

## Requirements

* Ubuntu 22.04 recommended
* Python 3.10+
* Node.js 20+
* Git
* Internet connection for Gemini authentication or API access

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/chomi14/ros-mcp-server.git
cd ros-mcp-server
```

### 2. Create and activate a Python virtual environment

```bash
python3 -m venv venv_ros_mcp
source venv_ros_mcp/bin/activate
pip install --upgrade pip
```

### 3. Install project dependencies

Using `uv`:

```bash
pip install uv
uv sync
```

If `uv sync` is not preferred, try:

```bash
pip install -e .
```

### 4. Install Gemini CLI

```bash
npm install -g @google/gemini-cli
```

### 5. Authenticate Gemini CLI

#### Option A: Browser login

```bash
gemini
```

#### Option B: API key

```bash
export GEMINI_API_KEY=YOUR_API_KEY
gemini
```

To make the API key persistent:

```bash
echo 'export GEMINI_API_KEY=YOUR_API_KEY' >> ~/.bashrc
source ~/.bashrc
```

---

## Running the Server

Activate the Python environment and run the MCP server:

```bash
cd ~/ros-mcp-server
source venv_ros_mcp/bin/activate
python server.py
```

---

## Typical Workflow

### 1. Start the MCP server

```bash
cd ~/ros-mcp-server
source venv_ros_mcp/bin/activate
python server.py
```

### 2. Open another terminal and start Gemini CLI

```bash
gemini
```

### 3. Enter a natural-language command

Examples:

* `Execute the first machining pass using ~/ros2_ws/data/ep/real_flat.txt`
* `After machining, run the filtering pipeline using the collected raw data`
* `Run the analysis pipeline and summarize the results`

---

## Notes

* This repository should be used together with the ROS 2 workspace repository.
* Actual robot-side execution is performed in `ros2_ws`.
* Before issuing real execution commands, make sure the ROS 2 workspace is built and sourced.
* If microphone/STT features are used, additional audio-related dependencies may be required depending on the system environment.

---

## Related Repository

* ROS 2 workspace: [ros2_ws](https://github.com/chomi14/ros2_ws)

---

## Troubleshooting

### Gemini CLI is not recognized

Check installation:

```bash
gemini --version
node -v
npm -v
```

### Python package import errors

Make sure the virtual environment is activated:

```bash
source venv_ros_mcp/bin/activate
```

Then reinstall dependencies if necessary:

```bash
pip install uv
uv sync
```

### Server starts but commands do not trigger ROS execution

Check the following:

* the ROS 2 workspace is available
* the ROS environment has been sourced
* required ROS nodes/services are running
* file paths are valid
* the companion workspace is correctly configured

### Microphone/STT does not work

Check:

* microphone connection
* OS audio settings
* required audio libraries and permissions

---

## Author

Jun

````

---

# `ros2_ws/README.md`

```markdown
# ros2_ws

## Overview
`ros2_ws` is the ROS 2 workspace for the actual machining / polishing pipeline.  
This repository contains the ROS-side execution environment for:
- trajectory/path execution
- raw data collection
- filtering / preprocessing of collected data
- analysis of execution results

This workspace is intended to be used together with the companion repository:
- `ros-mcp-server`: natural-language / MCP interface layer
- `ros2_ws`: actual ROS 2 execution and analysis workspace

---

## Environment
- Ubuntu 22.04 recommended
- ROS 2 Humble
- Python 3.10+
- colcon

---

## Workspace Structure

```text
ros2_ws/
├── src/                    # ROS 2 packages
├── data/                   # input paths, raw logs, processed data, analysis outputs
├── build/                  # generated by colcon build (not tracked)
├── install/                # generated by colcon build (not tracked)
├── log/                    # generated by colcon build (not tracked)
└── README.md
````

---

## Build

### 1. Source ROS 2

```bash
source /opt/ros/humble/setup.bash
```

### 2. Move to the workspace

```bash
cd ~/ros2_ws
```

### 3. Build the workspace

```bash
colcon build
```

### 4. Source the built workspace

```bash
source install/setup.bash
```

---

## Basic Usage

Before running nodes, source the environments:

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
source install/setup.bash
```

Then launch the required node(s) depending on the experiment or machining setup.

Example:

```bash
ros2 launch <your_package> <your_launch_file>.launch.py
```

Or:

```bash
ros2 run <package_name> <node_name>
```

---

## Typical Pipeline

A standard workflow is:

1. Prepare an input path / trajectory file
   Example:

   ```bash
   ~/ros2_ws/data/ep/real_flat.txt
   ```

2. Execute the first machining pass

3. Save the collected raw execution data

4. Run the filtering pipeline using the collected raw data

5. Run the analysis pipeline and inspect the summary results

---

## Data Flow

### Input

* planned path / trajectory file (`.txt`)

### Intermediate / execution output

* raw execution logs
* robot or simulation output data

### Processed output

* filtered velocity / force / trajectory data
* processed logs for evaluation

### Final output

* analysis summaries
* comparison results
* metrics / plots / reports

---

## Practical Example

### Build and source the workspace

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
colcon build
source install/setup.bash
```

### Example input

```bash
~/ros2_ws/data/ep/real_flat.txt
```

### Example command flow

1. Execute a machining pass using the prepared input path
2. Collect raw data from the run
3. Run filtering on the collected raw data
4. Run analysis and inspect the summary output

---

## Integration with `ros-mcp-server`

This workspace can be controlled through the `ros-mcp-server` repository, which provides a natural-language interface via Gemini CLI / MCP.

Typical integrated workflow:

1. Build and source this workspace
2. Start required ROS nodes / services
3. Start `ros-mcp-server`
4. Send commands through Gemini CLI

---

## Important Notes

* Do **not** commit `build/`, `install/`, or `log/`.
* Large raw logs, bag files, temporary outputs, and generated results should generally be excluded from version control unless they are intentionally shared samples.
* Always source both the ROS 2 environment and the workspace setup file before running nodes.
* File paths used in commands should match the actual workspace structure on the machine.

---

## Related Repository

* MCP / Gemini interface: [ros-mcp-server](https://github.com/chomi14/ros-mcp-server)

---

## Troubleshooting

### `ros2` command not found

```bash
source /opt/ros/humble/setup.bash
```

### Package not found

Rebuild and source again:

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
```

### Build fails

Check:

* package dependencies
* Python environment
* missing system libraries
* package names and folder structure inside `src/`

### Execution works but analysis fails

Check:

* input/output file paths
* whether raw logs were successfully generated
* whether required Python scripts are available
* whether dependent packages are properly installed

---

## Author

Jun

````
