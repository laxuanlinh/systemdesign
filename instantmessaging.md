# Instant messaging

## Functional requirements
- 1-1, group and channel chat
- Type of media:
  - Text only
- Online users get notified, offline users get messages when they're back online
- Visible online/offline status

## Non-functional requirements
- Millions of daily users
- Most users are connected for 6-12 hours/day
- Hundreds of thousands of messages/second
- Each message is limited to 10k characters
- Channels/group chats can grow to hundreds of thousands of users
- 99.99% uptime
- Message delivery time <1000ms at 99pt
- Catching up on messages after connection <2000ms at 99pt

## System API design
- signup()
- login()
- get_homepage()
- user_search()
- send_message()
- create_channel()
- join_channel()
- send_message()

![Sequence of creating chat ID](create_chat_id_sequence.png "Sequence of creating chat ID")

![Create and send messages to channels](create_chat_id_sequence.png "Create and send messages to channels")

## System design
- Use WebSocket connections
- Idle WebSocket connections require very little RAM and CPU

### Send message flow
  ![Send message](send_message.png "Send message")
- When a user sends a message to `messaging service`, it sends request to `chat history service` for persistence and to `channel & group service` to get a list of subscribers of the channel or group
- Then the messaging service delivers the message to all connected users

### Get message history flow
  ![Get message history](get_message_history.png "Get message history")
- When a user comes back online, it establishes a connection to the `messaging service`
- At the same time, it also sends request to `message history service` to get lists of messages
  