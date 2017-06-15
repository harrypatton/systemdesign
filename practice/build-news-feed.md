# Source
* https://stackoverflow.com/questions/4162020/how-can-i-improve-this-php-mysql-news-feed
* https://www.quora.com/What-are-the-best-practices-for-building-something-like-a-News-Feed

# Areas
Main scenarios are below,

1. User can post status.
2. User can see news feed (from friends).

## Analysis
1. There're a few entities mentioned in scenarios: `users` and `feeds`.
2. A few relationships,
    * a user can post a feed
    * a user can add other users as friends
    * user can see feeds from other users.
3. Services
    * User profile management
    * User friends service
    * User post feed service
    * news feed service
 
 Base on #1 and #2, we can define the underlying data schema.
 
 * User table: store user profile information. UserId, name, gender.
 * User friends table: store user friend relationship table. UserId, FriendUserId.
 * Feed table: UserId, FeedId, FeedContent.
 
 Going back to #3, we would know which data used by each service,
 
 * User profile uses `User` table.
 * User friends service uses `User friends` table.
 * User post feed service uses `Feed` table.
 * News feed service service would use both `user friends` and `feed` table.
 
 ### Concerns
 1. The 4 services don't call other services yet. It uses the underlying DB directly. Is it right? 
    * one join to get all data including feed content?
    * one join to get candidate feed id list. Use another service to return feed content itself. This solution is better because
      * the backend can get a longer list of feed candidate and the front end can decide how many to show. When need to show, just get the content from cache based on the feed id.
      * the relationship table could have just two columns `userid` and `feedid`. The feed content can save in other DBs. In other words, we use this table to store feed metadata and another table or source to store the content itself.
