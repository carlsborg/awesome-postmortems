# awesome-postmortems
A collection of useful post-mortems of production issues in the wild

1. https://blog.github.com/2018-10-30-oct21-post-incident-analysis/
*Impact* : Degraded service for 24 hours and 11 minutes. Data was out of date and inconsistent.GitHub was also unable to serve webhook events or build and publish GitHub Pages sites. 
*How it went down*: Replacing a 100G optical equipment resulted in the loss of connectivity between US East Coast network hub and primary US East Coast data center. Connectivity restored in 43 seconds, during which failover mechanism failed over master to US West coast, but databases were now inconsistent, so could not failback to east coast. Strategy was to prioritize data integrity over site usability and time to recovery.
*Data Loss* :  Ultimately, no user data was lost
*Technologies*: MySQL, raft Consensus, Network partitions
