---
layout: page
title: "Report"
permalink: /report/
---

# Literature 
In the midst of Vision and Language Navigation innovation, AerialVLN was proposed, a UAV-based VLN task for outdoor environments specifically taking into consideration aerial actions as aerial navigation is significantly more complicated than ground-based VLN [1]. However, as energy awareness is not widely discussed in VLN, the typical path-length objective of existing approaches does not directly minimize energy consumption, nor allows constraining the energy of individual paths by battery capacity [2]. We will utilize a combination of Supervised Learning and Reinforcement Learning to better consider energy, building on top of their implementation.

Dataset Description
This Dataset includes AerialVLN and AerialVLN-S annotated images for training and evaluation, a total of 32.4GB.

Why These Dataset(s)

<ol type="A">
  <li>AerialVLN was selected because it is the only large scale outdoor aerial VLN dataset available. Its long flight paths and complex city level environments make fuel constraints a realistic and meaningful addition to the task. Without a dataset of this scale and complexity, testing energy aware navigation would not be possible.</li>
  <li>Why AerialVLN-S For most of our experiments we use AerialVLN-S since the shorter average path length makes training more computationally feasible while still capturing the core challenges of the task. It also has more evenly distributed path lengths which makes evaluation more consistent. </li>
</ol>


**Overview**
- Total size: 32.4GB
- Built using AirSim simulator on Unreal Engine 4
- 25 city level environments (cities, factories, parks, villages)
- 870+ different object types
- Supports dynamic weather (sun, rain, snow, fog) and lighting (morning, noon, night)


<img src="{{ '/images/a.png' | relative_url }}" width="650">
* recorded by AOPA licensed human UAV pilots to ensure realistic and high quality flight trajectories. Each path is paired with 3 instructions written by Amazon Mechanical Turk workers, giving the dataset a wide variety of ways to describe the same flight.


**Each Flight Data Set Contains**
- Natural language instructions describing landmarks and directional cues
- Flight trajectory as a sequence of 9 discrete actions: Move Forward, Turn Left, Turn Right, Ascend, Descend, Move Left, Move Right, Stop, and the initial starting pose
- RGB and depth images from a front facing camera at every step (depth range up to 100m)
- Semantic segmentation data available for future use

<img src="{{ '/images/b.png' | relative_url }}" width="650">

**Our Addition:**
Fuel Constraints - On top of the standard dataset, our preprocessing pipeline adds a fuel fraction at every step of every flight:
- Ranges from 1.0 (full tank) to 0.0 (empty)
- Computed dynamically during preprocessing
- Fed into the FuelGateModule during training
- Forces the model to adjust its behavior based on remaining battery level
- This addition is unique to our work and is not part of the original AerialVLN dataset

**Dataset Linkzx:**
https://github.com/AirVLN/AirVLN 

**Problem:**
With the new constraints, specifically fuel capacity and limits, we focus on aerial navigation in the sky, UAV-based and aimed towards outdoor environments.

First step of our goal is to implement fuel constraints on our first method, Teacher Forcing, and to compare the performance of simulation without fuel constraints. This method is primarily going to be used as a **baseline**.  


**Motivation:**
Many existing VLN tasks are built for agents that navigate on the ground, either indoors or outdoors. However, some tasks require intelligent agents to operate in the sky, such as UAV-based goods delivery, traffic/security patrol, and scenery tours [1]. Most importantly, aerial navigation is a field with much work to be done.


# Methods
**Data Preprocessing Methods:**
The preprocessing converts the raw simulation data from the AirSim environment into a high-dimensional feature space. We distinguish between the baseline architectural encoding provided by AirVLN and our novel Fuel-Aware state implementation. 

**Baseline preprocessing:**
To manage the computational load of the AerialVLN dataset, the baseline processes the raw sensor data into optimized feature vectors using a two-stream encoding process:
- Visual Feature Extraction: The raw RGB and Depth maps are passed through a pretrailed convolutional neural network. This is done by stripping away the actual images' spatial data and only saving the final feature vectors, representing the semantic content of the UAV images. 
- Language Tokenization: The raw textual navigation instructions are processed using a tokenizer to convert natural language into numerical tokens, mapping the human readable commands into the model’s embedding space.
- Serialized packaging: These features are synchronized and packaged into high-performance binary database files to maximize the throughput during the Teacher Forcing (TF) training loop.
  
**Dynamic Fuel Constraining:**
To introduce energy awareness into the navigation task, we implemented a dynamic preprocessing layer that injects space dependent constraints on the fly:
- Fuel fraction: For every discrete step within the flight trajectory, we calculate a normalized fuel fraction representing the battery life
- Mathematical formulation: This fraction is derived as the quotient of remaining mission steps over the total allocated steps: Fuel fraction[0.0, 1.0] = (Remaining steps)/(Maximum steps)
- State injection: This scalar value is appended to the visual and linguistic features at every timestep, allowing the model to perceive the remaining “survival time” relative to the goal state.

## ML Algorithms and Models
The primary model utilized up to this point is Teacher Forcing. Teacher forcing is a supervised learning algorithm that is useful in sequential models, such as navigation problems. It trains the model by taking the ground-truth as the input to the next step instead of using the predicted action. This allows the model to maintain stability, avoiding accumulation of error. 

In the original implementation of AerialVLN, a dataset of pre-planned trajectories represents the ground-truth, including expected camera data, navigation instructions, and actions. It attempts up to a maximum action limit. At each step, it passes the RGB and depth observations through the Seq2Seq policy, along with the previous ground-truth action and current hidden state of the RNN, to output a probability distribution of actions. The primary training objective is an imitation loss, where the trainer computes the primary loss by comparing the network’s predicted actions against the ground-truth action. Thus, Teacher Forcing encourages the model to imitate the expert trajectories. 

**Fuel Constraints**

Our extension of the model involves the incorporation of fuel constraints. We introduced fuel awareness as an additional input to guide the decision making process. As discussed in the pre-processing section, the fuel constraint is defined as the ratio of steps remaining to the maximum steps: it is a continuous variable from 0.0 to 1.0, where 0.0 represents no fuel remaining. 

Remaining Steps / Max Steps = Fuel Fraction [0.0 - 1.0]

The fuel parameter is fed into the present model and the gate module, which determines the importance of the fuel fraction. In addition to the original standard action loss, we also calculate an auxiliary loss using MSE, which trains the Fuel Budget Predictor for future steps. This auxiliary loss is in as a specialized component called the FuelGateModule, which essentially keeps the gate high or low depending on if the fuel is high or empty. The gate includes features that are important for exploration and goal-oriented navigation, so if fuel is limited, the drone will shift towards more direct trajectories instead of exploring the environment. 

Overall, the objective of this fuel awareness constraint, in addition to the original imitation loss, is to adapt the model’s predictions to be aware of physical constraints. In the beginning, when fuel is abundant, the model will prioritize exploration and improve familiarity with new environments. When fuel is low, the model will shift towards efficient paths to maximize likelihood of reaching the end within budget. 









