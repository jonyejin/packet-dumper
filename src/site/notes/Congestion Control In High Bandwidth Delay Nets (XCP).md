---
{"dg-publish":true,"permalink":"/congestion-control-in-high-bandwidth-delay-nets-xcp/","tags":["PaperReview","CSCI651Papers","ComputerNetworks"]}
---

# Takeaways

This is the famous `XCP` paper! `Explicit Control Protocol` - components in end-points _and_ routers. 
Goal: 
1. separate fairness from congestion control
2. and do this without per-flow state in the router
Aiming for:
- high utilization
- good fairness
- low queueing delay (like RED and CoDel and BBR)

# Some Notes
current status:
– 2002 SIGCOMM paper was just simulations

– ~2004 ISI and others implemented it and examined IETF standards
• some slides and much discussion from Aaron Falk and Ted Faber at ISI

– ~2008 interest waned
