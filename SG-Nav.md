
# SG-Nav: Online 3D Scene Graph Prompting for LLM-based Zero-shot Object Navigation
Arxiv: https://arxiv.org/pdf/2410.08189
# Summary

Uses Scene Graph with LLM for navigation. It is a frontier based exploration method, where the agent finds the object goal in a zero shot manner. Only object category is provided to the agent. It build scene graph incrementally, also builds the occupancy map. LLM is then given scene graph and prompted to tell the frontier, where the object goal is most likely to be seen. 
# Introduction

Obejct Navigation: Requires an agent to navigate to an object specified by its category in unknown environment.
Previous modular methods use Semantic map and RL-based policy for navigation. However, these method requires time-consuming training in simulation environments, which don't generalize well to new environment and new object categories. RL methods have low sample efficiency.

##### Zero Shot Setting:
System does not require any training or finetuning before applied to real-world scenarios, and the goal category can be freely specified by text in an open-vocabulary manner.

## Questions

Q. What does frontier mean in navigation?
Ans: From Gemini:  **frontier** is the boundary between **known open space** and **unexplored territory**
### How Frontiers Work

When a robot builds a map (like an occupancy grid), it categorizes cells into three states:

1. **Free:** Areas the robot has seen and confirmed are empty.
    
2. **Occupied:** Areas where sensors detected an obstacle (walls, furniture).
    
3. **Unknown:** Areas the sensors haven't reached yet.
    

A **frontier** is specifically any "Free" cell that is right next to an "Unknown" cell.


## Paper Summary

Most likely this is what is happening for navigation
### The Iterative Loop

Because the environment is initially unknown, the agent follows this recursive flow:

1. **Selection (The LLM):** The LLM looks at the _current_ Scene Graph and the available **frontiers**. It selects the most promising frontier that might lead to the goal (e.g., "The goal is a mug; mugs are in kitchens; go to the frontier near that cabinets-like structure").
    
2. **Navigation (The Controller):** The agent moves toward that frontier.
    
3. **Expansion (The Sensor):** As it reaches the frontier, it "sees" new space. It detects new objects and updates the **3D Scene Graph**.
    
4. **Verification (The Re-Perception):** * **If the goal is found:** The agent checks if the object's "confidence score" is high enough. If yes, the task is complete.
    
    - **If the goal is NOT found:** The agent generates **new frontiers** based on the newly expanded map.
        
5. **Repeat:** The LLM receives the updated graph and makes a _new_ decision.

#### Scene Graph Creation

- Scene graph creation process is same as [[ConceptGraphs]] paper.
- Edge generation process is different. Also they incrementally generate the scene graph with edges, instead of [[ConceptGraphs]] which generates them in one go.
- They have room, groups, and object categories. For kitchen, we probably don't need room and groups?
- They say to incrementally add edges they have an optimized algorithm which works in linear time, linear in number of new nodes. Suppose $L_{pro}$ and $L_{res}$ represent the number of tokens of the prompt and LLM’s response respectively. $m$ is newly registered nodes, $n$ previous nodes

$$
\frac{T_{our}}{T_{naive}} = \frac{(L_{pro} + m(m+n) \cdot L_{res} + m(m+n) \cdot 2)^r}{m(m+n) \cdot (L_{pro} + L_{res})^r} < \frac{c}{m+n}
$$

#### Hierarchical Chain-of-Thought Prompting

- We compute the probability $P^{fro}$ of each frontier by prompting LLM with our 3D scene graph. 
- At time t, we divide the scene graph Gt into subgraphs, each of which is determined by an object node with all its parent nodes and other directly connected object nodes. 
- For each subgraph, we predict $P^{sub}$, the possibility of goal appearing near this subgraph. Then the probability $P_{i}^{fro}$ of the $i$-th frontier can be averaged by
$$
P_{i}^{fro} = \sum_{j=1}^{M} \frac{P_{j}^{sub}}{D_{ij}} \quad (i \in \{1, 2, \dots, N\})
$$
- where $M$ is the total number of subgraphs and $D_{ij}$  is the distance between the center of the $i$-th frontier and the central object node of the $j$-th subgraph

Sequence of prompts
- Predict the most likely distance between [object] and [goal],
- Ask questions about the [object] and [goal] for predicting their distance
- Given [nodes] and [edges] of subgraph, answer above [questions].
- Based on the above conversation, determine the most likely distance between this subgraph and [goal]
-  $P_{sub}$ can be simply computed by taking the inverse of the outputted distance

#### Graph Based Re-perception

- When the agent detects out a goal object, it will approach to this object, observe it from multiple perspectives and accumulate a credibility score $S$
- For the $k$-th RGB-D observation since this moment, we compute the credibility of it by
$$
 S_{k} = C_{k} \cdot \sum_{j=1}^{M} \frac{P_{j}^{sub}}{D_{j}}
$$
- where $C_{k}$ is the confidence score of this goal object, $D_{j}$ is the distance between the central object node of the $j$-th subgraph and this object
- The success condition for our re-perception mechanism:
$$
N_{stop} < N_{max}, \text{ where } N_{stop} = N, \text{ s.t. } \sum_{i=1}^{N-1} S_{i} < S_{thres} \le \sum_{i=1}^{N} S_{i}
$$
- where Nmax is a pre-defined maximum number of steps. 
- If succeed, the agent will navigate to this goal object without any further exploration or perception. 
- Otherwise, it will give up this object and continue the exploration

