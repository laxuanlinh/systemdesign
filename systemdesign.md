# System design

## Step-by-step process
- Gathering functional requirements
- Gathering non-function requirements
- Defining system's API and sequence of events
- Designing for functional requirements
- Addressing non-functional requirements

## Scalable image sharing social media platform
### Functional requirements:
- What info of users to store?
- What type of media can be shared? Text? Image? Video?
- What type of relationship between users?
- What operations can users perform on the platform?

`In scope`:
- When a user register, they provide:
  - Name, email, password, profile image
  - Optionally age, location
- Users can share only images
- Unidirectional relationship
- A user can:
  - Upload image
  - Search for other users
  - Follow or unfollow
  - View other users' profile, shared images
  - View newsfeed

`Out of scope`:
- Other formats like text, video
- Reaction, commenting ...

### Non-functinal requirements:
- Scalability:
  - Billion of users
  - Hundreds of millions visits/day
  - Each user uploads 1 image/day
  - Each image size ~2MB
  - Data processing volume ~1PD/day
- Availability:
  - 99.99% 
- Performance:
  - 500ms at 99 percentile
  - Timeline load time <1000ms at 99 percentile

### API design:
- Register, login and follow sequence:

  ![Register, login and follow sequence](image_sharing_login_follow_sequence.png "Register, login and follow sequence")
- View newsfeed:

  ![Register, login and follow sequence](image_news_feed_sequence.png "Register, login and follow sequence")
- System's APIs based on the sequence diagrams:
  - `get_homepage()`
  - `register()`
  - `login()`
  - `post_image()`
  - `search_user()`
  - `follow()`
  - `get_timeline()`

- `User service` is to save the user details to the database. 
- The user database should be documnet No SQL database due to the flexibility of adding new fields in the future.
- The user profile photos are stored in an `object store` and the users database only stores the URL.
- The `post service` and `post database` is to store the posts from users, the `post database` can simply be a SQL database with all the images stored in the `object store` and the `post database` only stores the URLs of a post.
- For searching, we should have a dedicated `search service` with text-based search engine like Elastic search and a document database
- To keep user `search service` and the `user service` in sync, we have a message broker in the middle so that every time an user is updated, `user service` sends an event to the broker and the `search service` can synchronize
- To store the relationship between 2 users, we can either use a `follow service` which is more complicated, or create a collection in the `user database` to store user IDs of the follower and the followee.
- When a user get home page, we can send a request to `user service` to get all users, then send another request to `post service` to get the latest posts from these users. This approach is not efficient.
- We can have a `timeline service` and a key/value database to cache the latest posts of a user. When post service inserts a new post, the `timeline service` updates asynchronously, pushes the new post in and the old post out. One of the issue is that the latest posts don't appear immediately on users' timelines and can only be eventually consistent but it's completely acceptable.
- We have implemented `CQRS pattern` when separate the read and write operations to user and `post database`.
- We also use `materialized view patterns` to optimize the post query time and avoid complex SQL query to the `post database`
  ![Functionality](image_share_functionality.png "Functionality")