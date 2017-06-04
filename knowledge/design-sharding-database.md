# Problem
Source: https://www.interviewbit.com/problems/sharding-a-database/

Let's design a sharding scheme for key-value storage.

## Scenarios
List all features we should support and prioritize them. Clarify the requirements

  * total data size: 100TB
  * will keep growing? yes, 1TB every day.
  * machine config: 72G RAM and 10TB disk.
  * is key-value evenly distributed? yes.
  * is each key independently? yes
    
## Needs
