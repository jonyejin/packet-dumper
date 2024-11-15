---
{"dg-publish":true,"dg-path":"Controlling Queue Delay - Nichols and Jacobson (2012).md","permalink":"/controlling-queue-delay-nichols-and-jacobson-2012/","tags":["PaperReview","CSCI651Papers"]}
---

# Takeaways
- Assumption: routers can measure  entrance and leaving time for each packet. (this is called sojourn time)
- Goal: As router, we want to do something if we observe buffer bloat or queue too full. (Which is called Active Queue Management)
	* But how to distinguish between `just full` and `bufferbloat(standing queue)`?
		* For each flow, good queue should be resolved in one RTT (from sender to receiver) If it is not cleared in one RTT - it is bad queue.
		* (I think routers use packet timestamp to measure RTT for each flow.)
			* one RTT means `propagation time` (+queueing time), also the time we get the delay to get the response for the request. This means that we can get the feedback for the request we sent. If we act early - congestion control may take less than 1 RTT. 
		* We look at minimum queue size over last RTTs. If it is same - which means sender did not react to slow down the sending - there is buffer bloat.
		* How to measure queue size? Of course we can track the whole length of queue, but using `sojourn time` is more cheaper? (My guess) - cause we can see only the first packet in the queue. 
		* If sojourn time is `> 5ms(default target)` : congestion.



> [!related] Related Documents
> 

