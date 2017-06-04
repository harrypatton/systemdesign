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

Note: I don't feel scenario is very clear in this case. For now let's follow the source article.

## Needs
1. How frequently do we have to add machine? every 10 days (1TB grow every day).

## Deep Dive
1. Can we have a fixed number of shard? No, data keeps growing. One shard may explode later.
2. How many shards do we need? 100TB / 10TB = 10 for now.
3. How to distribute the data? use consistent hashing. naive way (mod shard count) is not efficient.
