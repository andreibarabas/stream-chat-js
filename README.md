# Stream Chat JS

[![NPM](https://img.shields.io/npm/v/stream-chat.svg)](https://www.npmjs.com/package/stream-chat)

stream-chat-js is the official JavaScript client for Stream Chat, a service for building chat applications.

You can sign up for a Stream account at <https://getstream.io/chat/get_started/>.

## Installation

### Install with NPM

```bash
npm install stream-chat
```

### Install with Yarn

```bash
yarn add stream-chat
```

### Using JS deliver

```html
<script src="https://cdn.jsdelivr.net/npm/stream-chat"></script>
```

## API Documentation

Documentation for this JavaScript client are available at the [Stream Website](https://getstream.io/chat/docs/?language=js).

### Typescript (v2.x.x)

The StreamChat client is setup to allow extension of the base types through use of generics when instantiated. The default instantiation has all generics set to `Record<string, unknown>`.

```typescript
StreamChat<{
  attachmentType: AttachmentType;
  channelType: ChannelType;
  commandType: CommandType;
  eventType: EventType;
  messageType: MessageType;
  reactionType: ReactionType;
  userType: UserType;
}>
```

Custom types provided when initializing the client will carry through to all client returns and provide intellisense to queries.

**NOTE:** If you utilize the `setAnonymousUser` function you must account for this in your user types.

```typescript
import { StreamChat } from 'stream-chat';
// or if you are on commonjs
const StreamChat = require('stream-chat').StreamChat;

type ChatChannel = { image: string; category?: string };
type ChatUser1 = { nickname: string; age: number; admin?: boolean };
type ChatUser2 = { nickname: string; avatar?: string };
type UserMessage = { country?: string };
type AdminMessage = { priorityLevel: number };
type ChatAttachment = { originalURL?: string };
type CustomReaction = { size?: number };
type ChatEvent = { quitChannel?: boolean };
type CustomCommands = 'giphy';

type StreamType = {
  attachmentType: ChatAttachment;
  channelType: ChatChannel;
  commandType: CustomCommands;
  eventType: ChatEvent;
  messageType: UserMessage | AdminMessage;
  reactionType: CustomReaction;
  userType: ChatUser1 | ChatUser2;
};

// Instantiate a new client (server side)
// you can also use `new StreamChat<T,T,...>()`
const client = StreamChat.getInstance<StreamType>('YOUR_API_KEY', 'API_KEY_SECRET');

/**
 * Instantiate a new client (client side)
 * Unused generics default to Record<string, unknown>
 * with the exception of Command which defaults to string & {}
 */
const client = StreamChat.getInstance<StreamType>('YOUR_API_KEY');
```

Query operations will return results that utilize the custom types added via generics. In addition the query filters are type checked and provide intellisense using both the key and type of the parameter to ensure accurate use.

```typescript
// Valid queries
// users: { duration: string; users: UserResponse<ChatUser1 | ChatUser2>[]; }
const users = await client.queryUsers({ id: '1080' });
const users = await client.queryUsers({ nickname: 'streamUser' });
const users = await client.queryUsers({ nickname: { $eq: 'streamUser' } });

// Invalid queries
const users = await client.queryUsers({ nickname: { $contains: ['stream'] } }); // $contains is only an operator on arrays
const users = await client.queryUsers({ nickname: 1080 }); // nickname must be a string
const users = await client.queryUsers({ name: { $eq: 1080 } }); // name must be a string
```

**Note:** If you have differing union types like `ChatUser1 | ChatUser2` or `UserMessage | AdminMessage` you can use [type guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types) to maintain type safety when dealing with the results of queries.

```typescript
function isChatUser1(user: ChatUser1 | ChatUser2): user is ChatUser1 {
  return (user as ChatUser1).age !== undefined;
}

function isAdminMessage(msg: UserMessage | AdminMessage): msg is AdminMessage {
  return (msg as AdminMessage).priorityLevel !== undefined;
}
```

Intellisense, type checking, and return types are provided for all queries.

```typescript
const channel = client.channel('messaging', 'TestChannel');
await channel.create();

// Valid queries
// messages: SearchAPIResponse<ChatAttachment, ChatChannel, CommandTypes, UserMessage | AdminMessage, CustomReaction, ChatUser1 | ChatUser2>
const messages = await channel.search({ country: 'NL' });
const messages = await channel.search({ priorityLevel: { $gt: 5 } });
const messages = await channel.search({
  $and: [{ priorityLevel: { $gt: 5 } }, { deleted_at: { $exists: false } }],
});

// Invalid queries
const messages = await channel.search({ country: { $eq: 5 } }); // country must be a string
const messages = await channel.search({
  $or: [{ id: '2' }, { reaction_counts: { $eq: 'hello' } }],
}); // reaction_counts must be a number
```

Custom types are carried into all creation functions as well.

```typescript
// Valid
client.connectUser({ id: 'testId', nickname: 'testUser', age: 3 }, 'TestToken');
client.connectUser({ id: 'testId', nickname: 'testUser', avatar: 'testAvatar' }, 'TestToken');

// Invalid
client.connectUser({ id: 'testId' }, 'TestToken'); // Type ChatUser1 | ChatUser2 requires nickname for both types
client.connectUser({ id: 'testId', nickname: true }, 'TestToken'); // nickname must be a string
client.connectUser({ id: 'testId', nickname: 'testUser', country: 'NL' }, 'TestToken'); // country does not exist on type ChatUser1 | ChatUser2
```

## More

- [Logging](docs/logging.md)
- [User Token](docs/userToken.md)

## Contributing

We welcome code changes that improve this library or fix a problem, please make sure to follow all best practices and add tests if applicable before submitting a Pull Request on Github. We are very happy to merge your code in the official repository. Make sure to sign our [Contributor License Agreement (CLA)](https://docs.google.com/forms/d/e/1FAIpQLScFKsKkAJI7mhCr7K9rEIOpqIDThrWxuvxnwUq2XkHyG154vQ/viewform) first. See our license file for more details.

### Commit message convention

Since we're autogenerating our [CHANGELOG](./CHANGELOG.md), we need to follow a specific commit message convention.
You can read about conventional commits [here](https://www.conventionalcommits.org/). Here's how a usual commit message looks like for a new feature: `feat: allow provided config object to extend other configs`. A bugfix: `fix: prevent racing of requests`.

## Release (for Stream developers)

Releasing this package involves two GitHub Action steps:

- Kick off a job called `initiate_release` ([link](https://github.com/GetStream/stream-chat-js/actions/workflows/initiate_release.yml)).

The job creates a pull request with the changelog. Check if it looks good.

- Merge the pull request.

Once the PR is merged, it automatically kicks off another job which will create the tag and created a GitHub release.

## We are hiring

We've recently closed a [$38 million Series B funding round](https://techcrunch.com/2021/03/04/stream-raises-38m-as-its-chat-and-activity-feed-apis-power-communications-for-1b-users/) and we keep actively growing.
Our APIs are used by more than a billion end-users, and you'll have a chance to make a huge impact on the product within a team of the strongest engineers all over the world.

Check out our current openings and apply via [Stream's website](https://getstream.io/team/#jobs).
