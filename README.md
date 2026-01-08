# 🏎️ F1 22 Reinforcement Learning Agent (PPO + Telemetry)

An autonomous racing agent trained to drive in **F1 22** using Deep Reinforcement Learning (PPO).

Unlike traditional vision-based bots that "see" pixels, this agent "feels" the car. It intercepts raw **UDP Telemetry packets** (Speed, G-Force, Wheel Slip, XYZ Coordinates) sent by the game engine and controls the car using a **Virtual Xbox 360 Controller**.

![F1 22 AI](https://img.shields.io/badge/Game-F1%2022-red)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Algorithm](https://img.shields.io/badge/Algorithm-PPO%20(Stable--Baselines3)-green)

## ✨ Key Features

*   **Telemetry-Based Observation:** Uses 60Hz UDP data for inputs (Speed, Grip, Position), resulting in 100x faster training than Computer Vision (CNNs).
*   **Virtual Lidar / GPS:** Implements a custom mapping system that compares the car's position to a recorded "Center Line," allowing the AI to "see" corners 150m ahead.
*   **Analog Control:** Uses `vgamepad` to simulate an Xbox controller for smooth steering and throttle application (continuous action space).
*   **High-Speed Training:** Optimized Python synchronization allows training at **3x - 5x Game Speed** using Cheat Engine speedhacks.
*   **Robust Auto-Reset:** Includes macros to automatically navigate game menus and restart the lap upon crashing.

## 🛠️ Prerequisites

### 1. Software
*   **F1 22** (Game)
*   **Python 3.10 or 3.11** (Recommended)
*   **[ViGEmBus Driver](https://github.com/nefarius/ViGEmBus/releases)** (Required for the Virtual Controller to work on Windows).

### 2. Game Settings (Critical!)
You must configure F1 22 exactly as follows for the code to read data correctly:
*   **Options -> Settings -> Telemetry Settings**:
    *   **UDP Telemetry:** `On`
    *   **UDP Broadcast Mode:** `Off`
    *   **UDP IP Address:** `127.0.0.1`
    *   **UDP Port:** `20777`
    *   **UDP Send Rate:** `60Hz`
    *   **UDP Format:** `2022`
    *   **Your Telemetry:** `Public`

## 📦 Installation

1.  Clone the repository:
    ```bash
    git clone https://github.com/sting-raider/f1-rl-agent.git
    cd f1-rl-agent
    ```

2.  Create a virtual environment (Optional but recommended):
    ```bash
    python -m venv venv
    .\venv\Scripts\activate
    ```

3.  Install Python dependencies:
    ```bash
    pip install gymnasium stable-baselines3[extra] shimmy>=2.0 numpy vgamepad tensorboard
    ```
    *Note: If you have an NVIDIA GPU, install the CUDA version of PyTorch manually:*
    ```bash
    pip install torch --index-url https://download.pytorch.org/whl/cu121
    ```

## 🚀 How to Run

Open the `F1_AI_Trainer.ipynb` Jupyter Notebook.

### Step 1: Record the Track Map
*   Go to **Time Trial** mode in F1 22 (e.g., Bahrain or Austria).
*   Run the **"Track Recording"** cell in the notebook.
*   Drive **ONE** slow, clean lap staying exactly in the **center** of the track.
*   Stop the cell when you cross the finish line. This creates a `.pkl` map file.

### Step 2: Initialize Listeners
*   Run the **"Global UDP Listener"** cell **ONCE**. This starts the background thread that reads game data and creates the virtual controller.

### Step 3: Start Training
*   Run the **Training Loop** cell.
*   Click into the F1 22 game window.
*   The AI will take control. It will drive erratically at first.
*   **Monitor Progress:**
    ```bash
    tensorboard --logdir=logs
    ```

## 🧠 State Space & Rewards

### Observation Space (20 Inputs)
1.  **Car Physics:** Speed, Throttle, Steer, Brake, RPM, Gear.
2.  **Tires:** Wheel Slip for all 4 tires (Grip detection).
3.  **Navigation:** Lateral Deviation from center line, Normalized Lap Progress.
4.  **Lookahead:** X/Y coordinates of the track center at 20m, 50m, 100m, and 150m ahead (relative to car nose).

### Reward Function
The agent is rewarded for:
*   **Speed:** Proportional to `(Speed / 100)^2`.
*   **Progress:** Moving forward along the track.
*   **Centering:** Bonus for staying near the optimal line (Magnet).

The agent is penalized (and reset) for:
*   **Off-Track:** Going >12m from the center line.
*   **Stalling:** Speed dropping below 10 km/h.
*   **Invalid Lap:** Cutting corners (Game flag).

## ⚠️ Troubleshooting

**"Controller Disconnected"**
*   Do not restart the Jupyter Kernel repeatedly. The `global_gamepad` object must stay alive. Only run the initialization cell once per session.

**"Speed is 0.0"**
*   Check that F1 22 is **unpaused**.
*   Verify Game Settings: **UDP Format** must be `2022`.

**"Port Occupied"**
*   Restart the Jupyter Kernel to kill old background threads holding Port 20777.
