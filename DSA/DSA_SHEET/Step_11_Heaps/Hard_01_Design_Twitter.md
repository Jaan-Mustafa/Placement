# Design Twitter  🔴

**Reference:** [LeetCode 355 — Design Twitter](https://leetcode.com/problems/design-twitter/)

---

## Problem
Design a mini Twitter supporting `postTweet`, `getNewsFeed` (10 most recent tweets from the user and people they follow), `follow`, and `unfollow`.

```
postTweet(1, 5); getNewsFeed(1) -> [5]
follow(1, 2); postTweet(2, 6); getNewsFeed(1) -> [6, 5]
unfollow(1, 2); getNewsFeed(1) -> [5]
```

---

## Intuition
Stamp every tweet with a global increasing `time`. Store each user's tweets as `{time, tweetId}`. For the feed, gather the relevant users (self + followees) and use a **max-heap on timestamp** to merge their tweet streams, pulling the 10 most recent — exactly a k-way merge keyed by recency.

---

## Code (C++)
```cpp
class Twitter {
    int clock = 0;
    unordered_map<int, vector<pair<int,int>>> tweets;   // user -> {time, id}
    unordered_map<int, unordered_set<int>> following;   // user -> followees
public:
    void postTweet(int userId, int tweetId) {
        tweets[userId].push_back({clock++, tweetId});
    }
    vector<int> getNewsFeed(int userId) {
        priority_queue<pair<int,int>> mx;               // max-heap on time
        auto add = [&](int u) {
            for (auto& [t, id] : tweets[u]) mx.push({t, id});
        };
        add(userId);
        for (int f : following[userId]) add(f);         // self + followees

        vector<int> feed;
        while (!mx.empty() && feed.size() < 10) {       // 10 most recent
            feed.push_back(mx.top().second); mx.pop();
        }
        return feed;
    }
    void follow(int followerId, int followeeId) {
        if (followerId != followeeId) following[followerId].insert(followeeId);
    }
    void unfollow(int followerId, int followeeId) {
        following[followerId].erase(followeeId);
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | postTweet O(1); getNewsFeed O(T log T), T = candidate tweets |
| Space | O(users + tweets) |

> ⚠️ Don't let a user follow themselves (self-tweets are added separately). The default `priority_queue<pair<int,int>>` is a max-heap on `.first` (the timestamp), giving most-recent-first for free. For huge feeds, push only each user's latest tweet and advance lazily to keep the heap at O(followees).
