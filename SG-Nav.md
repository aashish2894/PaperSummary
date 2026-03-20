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

