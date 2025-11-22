[[Computer Architecture and Organization]]
# Introduction
 - A typical GPU does 36 trillion calculations every second
 - A GPU has over 10k cores, while a typical CPU will have around 24 cores. Each core of a CPU is very  powerful and can execute very complex instructions but capacity of doing parallel tasks is limited. a GPU while can handle super high concurrency, they can only do very basic computations. 

# Physical Architecture
- Has a large chip called GA102 (major area taken by processing cores)
- We have around 10k CUDA cores, 336 tensor cores and 84 ray tracing cores
- Cuda cores are simple binary calculators for simple calculations
- Tensor cores are matrix multiplication/addition and working with AI/ML models
- Ray tracing cores are specially made for running tracing algorithms.

## Different Families of GPUs
![[nvidia 30xx family.png]]
- All the nvidia 30xx families use the same GA102 chip. The main difference is, during manufacturing, we often come across defects in the chips. Engineers are able to isolate damaged chips from the rest of the circuitry. 
- Hence, the different models are essentially boards with some threshold % of damaged chips. a 3090ti will have almost all the cuda cores and other components in working order. 

### Inside a CUDA core
- It has around 410k transistors.
- A section of 50k transistors is responsible for performing `AxB + C` (FMA - fused multiply and add). Half of these cores implement this using 32 bit floating number (scientific notation) and the other half 
- Other sections perform subtraction, bitshifting and other queueing tasks. 
- It performs one multiply and 1 add addition per cycle. 
- So for a 3090 GPU, we get 2 calculations per core, 10k cores and 1.7 GHz clock speed, we get around 35.6 billion calculations per second. 
- It has special function units for trigonometry and other stuff but they are extremely few. 
### GDDR6 and GDDR7 memory
- When we start a game, the loading screen is effectively game dumping its models and scenes and other assets in the GPU. 
- A GPU has limited L2 cache which cannot load the whole game there, hence we have these GDDR6 memory (typically starting from 6GB) which keeps on continuously feeding the GPU with data. This memory doesnt transfer data in just binary but in multiple voltages to achieve higher bandwidth.

# Computational Architecture
## Embarrassingly parallel problem
- This refers to a class of problems where dividing a problem into parallel tasks is super easy. 
- Some examples of this classification include bitcoin mining, video game rendering etc. 
- This is done through [[Single Instruction Multiple Data]]



