[[Uber]] [[System Design]] [[Realtime Communication]]

- In initial stages, Uber used to make polling calls to ask for new data, i.e. requesting for new available drivers, knowing driver's new location etc.
- This lead to ~80% of requests to uber servers polling calls (and most of them returned no new data)
## Shift to Push based Communication
- The server pushes new data to the servers when the data is available. 
- For this, Uber created RAMEN (Realtime Asynchronous Messaging Network).
- For this they needed to work at their scale, they needed to know **what to push**, **when to push** and **how** to push that data to the client. 
### When to push - Fireball
- It decides to all the events and decides if its worth pushing this update to the client (events include - new location of a driver, driver accepting ride, rider requesting a request)
- If fireball thinks that this event needs to be pushed to client, it'll forward that event to Uber's API gateway

### What to Push - Uber's API Gateway
- Gateway will then now basis the event, will gather up the metadata that will be required with the associated event.
- It can gather this data from calling multiple services as it seems fit. 
- Then the API gateway will forward these requests to RAMEN

### How to push - RAMEN
- It was built on top of Server sent Events [[SSE]]. 
- It guaranteed atleast once delivery. 

## Hex Grid Based Spatial Partitioning
- There's still one big problem, how to know which drivers are near the rider who requested the ride?
- It becomes highly inefficient to query this when the number of active drivers is in millions.
- For this, Uber created [[H3]]. It's a hexagonal based spatial index. It divides the world into hexagons,.
- Any GPS co-ord can then be turned into a H3 index, and we can ask for drivers in a K-ring. `k=1` means, the first hexagonal circle covering your current H3 index.

## How did Uber Engineers made Mobile App super smooth on Unreliable Internet
### Edge Servers
- A simple win came in form of edge servers. 
- This means serving the request to a client from the server that is the nearest to the client. 
### Client Side Prediction of New Driver Locations
- Uber uses something called **dead reckoning**, i.e. trying to predict the new location of the driver given the last know speed and direction. 
- This is used along with [[Kalman Filters]] to get a smooth prediction of the next location.  
