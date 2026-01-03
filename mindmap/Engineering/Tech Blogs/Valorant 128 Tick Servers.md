# Objective
- Most of the online shooter games have some amount of peeker's advantage. Having a higher tick rate aims at reducing the peeker's advantage in online shooter games. 
- 128 tickrate means, processing 128 frames in one second, i.e., having an ability to process 1 frame in 7.8125ms. But having one CPU core handle 1 game is unrealistic and too expensive. 
- So the end goal was to have 3 games running on 1 cpu core on the backend rack.

> Let’s take that 7.8125ms, divide it by 3 gpc, and we end up with 2.6ms. But wait - we also need to reserve 10% for overhead of the OS, scheduling, and other software running on the host. After these calculations, we end up with a target budget of just 2.34ms per frame. When we looked at VALORANT’s initial data, we were at 50ms; we had a long way to go. This was going to be an effort that needed to involve the entire development team.

![Valorant FPS Guide](Assets/valorant_tick_guide.png)

# Application Bottlenecks
## Replication
- Property replication is state reconzilation mechanism in unreal engine 4 where it allows for network sync of state between server and clients.
- A dev can mark variable as replicated and it'll auto sync between the server and the client. 
- Drawbacks of this property is that this is pretty slow. On every frame, a lot of variables like player's health bar would be recalculated but in a game env, the health bar rarely changes. Multiply this by 10 players and a lot of more variables like ammo, money, armour etc, this quickly shoots up the time. 
- There's another networking tool available in unreal engine 4, the ability to make remote procedure calls. Using RFCs limit the scope of change on the frame that triggered the change. Eg, if a player is shot, a RFC call can be triggered on only that frame and the server will send this event to all the downstream clients. 

## Animation
To deduce whether the target was hit or not, the server side also needs to run almost the same set of animations of what the client sees. 

Hit registration works by saving player positions and animation state in a historical buffer. When server receives a packet of HitShot, it rewinds the player positions and animation state using the historical buffer to calculate if the shot hit or not. 

Two sets of optimizations were done to ensure more performant animations on the backend -
- Valorant team animated only every 4th frame on server side.
- Team saved on the amortized server performance by reducing animations in buy phases (a valo server running 150 games, around 50 of those games would be in the buy phase, not needed any sustantial animation needs)

After addressing these two and a few here and there bottlenecks, when running a benchmark, team noticed a blitzing 1.5ms to process a single frame but this number shot up to 5.7ms once 168 games were running on that single machine.

# COA Bottlenecks
[[Computer Architecture and Organization]]
## Inclusive L3 Cache policy
[[L1, L2 & L3 Caches]]
- Every core has its own L1 and L2 Cache. However, all of these cores share the L3 cache.
- Valorant suffered from inclusive cache problem that was prevalent on older intel xeon E5 processors. Upgrading from these chips to the new intel xeon scalable chips gave them a 30% performance boost.

## NUMA (Non Uniform Memory Access)
[[Non Uniform Memory Access (NUMA)]]
- In linux servers, it's pretty straightforward to work on NUMA-awareness. 
- While starting the process, team laucnhed the servers with a command like `numactl --cpunodebind={gameid % 2} --membind={gameid % 2} ShooterGameServe`
- This small change led to a boost of 5% and also led to more consistent performance across game servers on the node.

## C-States
- Another area of straightforward wins was in controlling the C-State that CPU entered.
- When a multi-core CPU runs, it allows cores to enter different power states. Under reduced load, cores will often enter lower states to conserve power. 
- However, once load increases, it takes time for those cores to swap to higher power states. 
- In highly cyclical workloads - like a bunch of game servers processing frames then sleeping - the CPU ends up swapping power states frequently. Each swap has a latency that negatively affects performance. 
- By limiting the processes to the higher C-States (C0, C1 and C1E), valorant devs were able to host another 1-3% games stably. It particularly stabilized performance of 60-90% loaded servers where the reduced workload was allowing many cores to frequently idle.

## Hyperthreading
[[Hyperthreading]]
- Valorant devs were able to squeeze around 25% performance improvements when turning on hyperthreading.

# Reference
- https://technology.riotgames.com/news/valorants-128-tick-servers
