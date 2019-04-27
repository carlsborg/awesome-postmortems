# awesome-postmortems
A collection of useful post-mortems of production issues in the wild


1.  **Summary**: Prod database partially wiped due to human error while trying to resync the secondary hot standby. No backups available because the daily backup process was not being run due to a versioning issue.
**Links**: https://about.gitlab.com/2017/02/10/postmortem-of-database-outage-of-january-31/  
**Impact** : GitLab.com service unavailable for many hours, lost 6 hours of prod user data.  
**How it went down**: Increased load on primary due to spam removal queries caused secondary to lag. 
[WAL segment archiving](https://www.postgresql.org/docs/9.3/continuous-archiving.html) was not enabled (see Concepts below), so secondary had to be manually resynced. This involves removing the data dir on the secondary, then running pg_basebackup to copy over the database from the primary to the secondary. Runbooks didn't document that pg_basebackup operations properly, engineer assumed it hung partially after creating some data, and then tries to wipe the data dir, but on the primary instead of the secondary. Backups were not available because the backup process was not running, because the DB was running 9.6, while pg_dump was 9.2 "When we went to look for the pg_dump backups we found out they were not there. The S3 bucket was empty, and there was no recent backup to be found anywhere."  
**Technologies**: Postgresql, pg_basebackup.  
**Concepts**: "... a running PostgreSQL system produces an indefinitely long sequence of Write Ahead Log records. The system physically divides this sequence into WAL segment files, which are normally 16MB apiece (although the segment size can be altered when building PostgreSQL). The segment files are given numeric names that reflect their position in the abstract WAL sequence. When not using WAL archiving, the system normally creates just a few segment files and then "recycles" them by renaming no-longer-needed segment files to higher segment numbers. It's assumed that segment files whose contents precede the checkpoint-before-last are no longer of interest and can be recycled."  



1.  **Summary**: Connectivity lost between datacenters, primary fails over but databases are inconsistent  
**Links**: https://blog.github.com/2018-10-30-oct21-post-incident-analysis/  
**Impact** : Degraded service for 24 hours and 11 minutes. Data was out of date and inconsistent.GitHub was also unable to serve webhook events or build and publish GitHub Pages sites.   
**How it went down**: Replacing a 100G optical equipment resulted in the loss of connectivity between US East Coast network hub and primary US East Coast data center. Connectivity restored in 43 seconds, during which failover mechanism failed over master to US West coast, but databases were now inconsistent, so could not failback to east coast. Strategy was to prioritize data integrity over site usability and time to recovery.  
**Technologies**: MySQL, raft Consensus, Network partitions  


1. **Summary**: Switchgear fails and locks out backup generator in the datacenter  
**Links**: https://youtu.be/uj7Ting6Ckk?t=1665, 
   https://www.popsci.com/science/article/2013-02/fyi-what-caused-power-outage-super-bowl  
**Impact** : Three unrelated incidents and impacts: Un-named American Airline lost $100M in revenue. Also Superbowl in 2013 was down for 34 minutes because of this exact same fault. AWS datacenter saw this exact same issue.  
**How it went down**: When the utility power fails, the switchgear starts up the generators, waits for a few seconds before for power to stabilize and then swings over. What goes wrong: if the fault looks like a short-to-ground inside the DC, the switchgear wont failover to the generators (because it can damage the generators). However turns out the short is usually not in the DC. The manufacturers refuse to disable this behaviour so AWS changed the firmware. Tradeoff is in the minority case where the generator gets damaged due to a short in the DC, they have backup generators ("redundant and concurrently maintainable".)  
**Technologies**: datacenter, switchgear, power supply, generators  
**Concepts**:  
  [Switchgear](http://www.academia.edu/1770072/Low_Voltage_Switchgear): A part of an electrical system that provides electrical isolation, switching and protection (against short circuits/overloads/overheating)  
  [Concurrent Maintainability](https://www.cibse.org/getmedia/fedd9b85-e8ea-44d0-a638-9c2a75f4ec4e/Transferable-Lessons-in-Tier-Based-Design.pdf.aspx)  "The ability for a system to be able to be maintained, fully and completely, while still being fully functional and available."  
  [Redundancy](https://www.cibse.org/getmedia/fedd9b85-e8ea-44d0-a638-9c2a75f4ec4e/Transferable-Lessons-in-Tier-Based-Design.pdf.aspx): Where the number of units, systems or subsystems, exceeds the minimum value required for capacity (N). Commonly this is defined as a function of N. (N+1, 2N, 2(N+1) etc)  
