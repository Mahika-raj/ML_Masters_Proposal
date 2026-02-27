# ML_Masters_Proposal


Report
Literature 

In the midst of Vision and Language Navigation innovation, AerialVLN was proposed, a UAV-based VLN task for outdoor environments [1]. However, as energy awareness is not widely discussed in VLN, the typical path-length objective of existing approaches does not directly minimize energy consumption, nor allows constraining the energy of individual paths by battery capacity [2]. We will utilize a combination of Supervised Learning and Reinforcement Learning to better consider energy.

Dataset Description
This Dataset includes AerialVLN and AerialVLN-S annotated images for training and evaluation, a total of 32.4GB. 

Dataset Link
https://github.com/AirVLN/AirVLN 

Problem
With the new constraints, specifically fuel capacity and limits, we focus on aerial navigation in the sky, UAV-based and aimed towards outdoor environments.

Motivation
Many existing VLN tasks are built for agents that navigate on the ground, either indoors or outdoors. However, some tasks require intelligent agents to operate in the sky, such as UAV-based goods delivery, traffic/security patrol, and scenery tours [1]. Most importantly, aerial navigation is a field with much work to be done.



Methods
Data Preprocessing Methods

3 Data Preprocessing methods we are considering are fuel tokenization, state-space normalization, and trajectory-based windowing:
Fuel Tokenization: Fuel capacity is a crucial constraint that is not quantified in the dataset. We can label trajectories with discrete tokens, such as low, middle, and high, or as a continuous function. The embedding of a fuel feature serves to steer towards both efficient and reliable path planning.
PyTorch Tokenizer
AutoTokenizer
State-space Normalization: Numerical data like position, cartesian and angular velocities, and fuel will be normalized into a standardized distribution so stable inputs get fed into the model. We want to balance them so that large-scale features like altitude do not dominate small-scale ones like our fuel capacity. 
Scikit-learn StandardScalar, RobustScalar
Trajectory-based Windowing: Instead of optimizing fuel efficiency with entire simulations in AirVLN, we can segment each simulation into windows: training each of the windows between landmarks in the city-level environments. This should make parallelization more optimized and leverage the dataset, especially since the AirVLN’s dataset is not extensive. 
PyTorch utils, torch.utils.data.DataLoader

ML Algorithms and Models 

We considered 2 supervised learning models: Recurrent Neural Network and Transformers. These would function by embedding the instructions, visuals, and fuel for inputs, and then outputting a low-cost trajectory. 

Alternatively, and perhaps more intuitively, we can pursue Proximal Policy Optimization (PPO), a reinforcement learning algorithm. Here, we would use fuel capacity as a Lagrange penalty.


Learning Methods 

Supervised Training:
Recurrent Neural Networks: 
Transformers: 
Likely both impl of PyTorch or Tensor

RL Method:
PPO with Lagrange Penalty: Fuel for drones can span a number of things, but the most common is LiPo batteries for quadcopters. There is a 2016 paper that explores energy-efficient drones and goes into the electrical “fuel” consumption for minimum-energy trajectories, so that will be a potentially useful reference [3]. The paper’s researchers leveraged ACADO, a MATLAB toolkit for KKT optimization, which serves as a proof of concept for PPO. The constraint for fuel could look something like:
L(x) = f(x) - λfuel gfuel(x) where gfuel(t) is an arbitrary constraint function for fuel remaining and x is the state space
Stable Baselines 3
Potential Results
We aim to create a model that increases the path smoothness by reducing the sharp turns it takes and reduces the fuel usage significantly, increasing the success rate when it is based on the amount of energy.

Discussion: 
This project extends aerial vision-and-language navigation by adding energy constraints, which makes the task more practical for real UAV use. By combining supervised and reinforcement learning, it explores whether navigation can stay effective while also reducing fuel use.


Citations:
[1]
Liu et al. (2023): AerialVLN: Vision-and-Language Navigation for UAVs (The primary AirVLN paper).

[2]
Pereira et al. (2025): Energy-Aware Coverage Path Planner for Multirotor UAVs (For the fuel modeling aspect).

[3]
Fabio Morbidi, Roel Cano, David Lara. Minimum-Energy Path Generation for a Quadrotor UAV. IEEE International Conference on Robotics and Automation, May 2016, Stockholm, Sweden. Ffhal01276199v2f

Other

Morbidi, F., Cano, R. & Lara, D. Minimum-energy path generation for a quadrotor UAV. In 2016 IEEE International Conference on Robotics and Automation (ICRA), 1492–1498 (2016). 
https://hal.science/hal-01276199/document 

Yacef, F., Rizoug, N., Degaa, L. & Hamerlain, M. Energy-efficiency path planning for quadrotor UAV under wind conditions. In 2020 7th International Conference on Control, Decision and Information Technologies (CoDIT), vol. 1, 1133–1138 (2020).
https://hal.science/hal-04504751/document 
