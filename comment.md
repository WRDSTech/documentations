Comment:

1. We need to reconsider the performance requirement. In our senario, this graph service handles large amount of data so the expectation is to achieve high throughput with acceptable latency.
2. We can estimate the theorectical limit under different network bandwidth. We also need to calculate the size of the graph given the number of nodes and edges.
3. By knowing the size of data transferred and the time to transfer the data under different network bandwidth, we can now define the range of acceptable latency and throughput.
4. We also need to consider how much data is need by the downstream ML service. Can that kind of data requirement be known in advance?
   1. If it can be know in advance, this service can prefetch the data and store it on some cache or objective storage
   2. If not, then
      1. Does the downstream service require all data at once?
         1. If so, it must tolerate some latency since it require time to query and transfer the data
         2. If not, it can accept data by streaming.
5. The availbility can be set to an empircal value. Maybe 90%？
6. For the visualizer, we can preload some data and store it on the objective storage
   1. In the overview mode, the nodes and edges are sampled. The sampled data is prepared in advance to ensure consistency.


Comment2

1. How to achieve your metrics
2. need to define what to monitor and the specific metric
   1. Figure it out during testing
3. deprioritize cache layer implementation
4. Initialization  phase, design phase not needed
5. 

SIT

1. specify what need to be tested
2. Websocket retry and rate limit

 Parking lot item:

1. is websocket needed?
   1. client determines the request rate
      1. how to smooth it out
      2. how to throttle
      3. how to scale
      4. who controls all of those?
   2. adds complexity
      1. becomes stateful
2. http streaming

SIT

1. Loading testing
2. provide rate limiting demo code


Trade off / Future work

1. stage 1, client base throttling
2. stage 2, stream based data transmission


Server side: 

1. ticket

Client:

1. periodical load testing
2. 


Update design doc

Asana Ticket Assignment
