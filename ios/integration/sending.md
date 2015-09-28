# Sending Messages
Once authenticated, users can send and receive messages, which are associated with a specific conversation object.

[LYRConversation](/docs/api/ios#lyrconversation) objects are created by calling `newConversationWithParticipants:options:error:`on [LYRClient](/docs/api/ios#lyrclient). By default, new conversations are set to be distinct, which ensures that there is only one unique conversation between the given set of users. Note: this feature was introduced in SDK version 0.14.0 and older versions of the SDK do not support the notion of distinct conversations (they are classified as "non-distinct"). The initialization variables are the following:

* `participants` - A mandatory array of user identifiers. As Layer Authentication allows you to represent users within the Layer service via your backend’s identifier for that user, a participant in a conversation is represented with that same user identifier.
* `options` - An optional dictionary of initialization options. You can initialize of metadata via the `LYRConversationOptionsMetadataKey` key, and you can specify if this conversation should or should not be distinct by using the `LYRConversationOptionsDistinctByParticipantsKey` key.
* `error` - An optional pointer to an error object whose value will be set if an error occurs.

```objectivec
// Creates and returns a distinct conversation object with the given user
NSError *error = nil;
LYRConversation *conversation = [layerClient newConversationWithParticipants:[NSSet setWithObjects:@"USER-IDENTIFIER", nil] options:nil error:&error];
```

```emphasis
Note, that it is not necessary to include the currently authenticated user in the participant array. They are implicit in all new conversations.***
```

If multiple users independently create distinct conversations with the same set of users, the server will automatically merge the conversations. This means that some properties of the conversation may change after it is created, but the Layer SDK will handle these changes for you.

You can also have multiple conversations with the same set of users, should you choose.

```
NSError *error = nil;
LYRConversation *topicA = [layerClient newConversationWithParticipants:[NSSet setWithObject:@"USER-IDENTIFIER"] options:@{LYRConversationOptionsDistinctByParticipantsKey : @(NO) } error:&error];
LYRConversation *topicB = [layerClient newConversationWithParticipants:[NSSet setWithObject:@"USER-IDENTIFIER"] options:@{LYRConversationOptionsDistinctByParticipantsKey : @(NO) } error:&error];
```


## Add/Remove Participants

Once a conversation has been created, participant lists remain mutable, meaning participants can be added or removed. The Layer service does not enforce ownership of conversations so any client can both add or remove participants from a conversation.

```objectivec
// Adds a participant to an existing conversation
// New participants will gain access to all previous messages in a conversation.
NSError *error = nil;
BOOL success = [conversation addParticipants:@[ @"USER-IDENTIFIER" ] error:&error];

// Removes a participant from an existing conversation
// Removed participants will only lose access to future content. They will retain access
// to the conversation and all preceding content.
NSError *error = nil;
BOOL success = [conversation removeParticipants:@[ @"USER-IDENTIFIER" ] error:&error];
```

```emphasis
Note, adding or removing participants from a distinct conversation removes the distinct status. For example, if you have the following conversations:

- Conversation A is distinct and has participants "1" and "2"
- Conversation B is distinct and has participants "1", "2", and "3"

Adding user "3" to Conversation A will result in the following:

- Conversation A is non-distinct and has participants "1", "2", and "3"
- Conversation B is distinct and has participants "1", "2", and "3"
```

## LYRMessagePart

Layer does not enforce restrictions on the type of data you send through the service. As such, [LYRMessagePart](/docs/api/ios#lyrmessagepart) objects are initialized with an `NSData` object and a MIME type string. The MIME type string simply describes the type of content the [LYRMessagePart](/docs/api/ios#lyrmessagepart) object contains.

The following demonstrates creating message parts with both `text/plain` and `image/png` MIME types.

```objectivec
// MIME type declarations
static NSString *const MIMETypeTextPlain = @"text/plain";
static NSString *const MIMETypeImagePNG = @"image/png";

// Creates a message part with a string of text and text/plain MIMEtype.
NSData *messageData = [@"Hi, how are you?" dataUsingEncoding:NSUTF8StringEncoding];
LYRMessagePart *messagePart = [LYRMessagePart messagePartWithMIMEType:MIMETypeTextPlain data:messageData];

// Creates a message part with an image
UIImage *image = [UIImage imageNamed:@"awesomeImage.png"];
NSData *imageData = UIImagePNGRepresentation(image);
LYRMessagePart *imagePart = [LYRMessagePart messagePartWithMIMEType:MIMETypeImagePNG data:imageData];
```

The [LYRMessagePart](/docs/api/ios#lyrmessagepart) object also declares a convenience method for creating messages with `text/plain` MIME Type.

```objectivec
// Creates a message part with a string of text
LYRMessagePart *part = [LYRMessagePart messagePartWithText:@"Hi, how are you?"];
```
<a name="warning"></a>
```emphasis
**NOTE**
While Layer does not place any restrictions on the MIME Type, Google and Apple dictate that the MIMEType string MUST conform to a "\*/\*" convention.  If the MIME Type does not contain a forward slash (/) you may have issues sending messages. For a comprehensive list of MIME Type values check out the [IANA's official registry of media types](http://www.iana.org/assignments/media-types/media-types.xhtml).
```

## LYRMessage

The [LYRMessage](/docs/api/ios#lyrmessage) object represents an individual message within a conversation. A message within the Layer service can consist of one or many pieces of content, represented by the [LYRMessagePart](/docs/api/ios#lyrmessagepart) object.

[LYRMessage](/docs/api/ios#lyrmessage) objects are initialized by calling `newMessageWithParts:options:error:` on [LYRClient](/docs/api/ios#lyrclient). This creates an [LYRMessage](/docs/api/ios#lyrmessage) object that is ready to be sent. The initialization variables are the following:

* `parts` - A mandatory array of message parts to be sent.
* `options` - An optional dictionary of initialization options. Currently, the only supported functionality is configuring Push Notification alert text and sounds via the `LYRMessageOptionsPushNotificationAlertKey` and `LYRMessageOptionsPushNotificationSoundNameKey` keys, respectively.
* `error` - A optional pointer to an error object whose value will be set if an error occurs.


```objectivec
// Creates and returns a new message object with the given conversation and array of message parts
NSError *error = nil;
LYRMessage *message = [layerClient newMessageWithParts:@[ messagePart ] options:nil error:&error];
```

```emphasis
**IMPORTANT**

By default, LayerKit will automatically download content for message parts whose content size is less that 2KB. If you want to send content larger than 2k like images or movies, please read the [Rich Content](/docs/ios/guides#richcontent) guide.
```

## Sending The Message

Once an [LYRMessage](/docs/api/ios#lyrmessage) object is initialized, it is ready to be sent. The message is sent by calling `sendMessage:error` on `LYRConversation`.

```objectivec
//Sends the specified message
NSError *error = nil;
BOOL success = [conversation sendMessage:message error:&error];
if (success) {
  NSLog(@"Message enqueued for delivery");
} else {
  NSLog(@"Message send failed with error: %@", error);
}
```

The `sendMessage:error` method returns a boolean value which indicates if the message has passed validation and was enqueued for delivery in the local data store. If LayerKit has a current network connection, the message will immediately be sent off of the device. Otherwise it will remain enqueued locally until the SDK re-establishes a network connection. At that point, the SDK will automatically send the message.

[LYRMessage](/docs/api/ios#lyrmessage) objects declare a boolean property, `isSent`, which tells your application if the message was successfully sent from your device and synchronized with the Layer service.

```emphasis
**Best Practice**

Conversations are not pushed to other participants, and are not queryable, until the first message is sent. Depending on your app's flow, you can rely on [distinct converesations](https://support.layer.com/hc/en-us/articles/204193200-Why-can-t-participants-query-a-newly-created-conversation-), or use a server to server connection with the [Platform API](/docs/platform#create-a-conversation) to create either distinct or non-distinct conversion which will be synced across all users on creation.
```

### Recipient Status

Layer declares 4 recipient statuses which allows applications to monitor the actual status of a message for every individual participants in a conversation. The states are the following:

* `LYRRecipientStatusInvalid` - The message status cannot be determined.
* `LYRRecipientStatusSent` - The message has successfully reached the Layer service and is waiting to be synchronized with recipient devices.
* `LYRRecipientStatusDelivered` - The message has been successfully delivered to the recipients device.
* `LYRRecipientStatusRead` - The message has been marked as read` by a recipient's device.

Applications can inspect recipient statuses by accessing the `recipientStatusByUserID` property on `LYRMessage` objects. The property is a dictionary wherein the keys are participant identifiers and the values are the current recipient status for that message. The following demonstrates inspecting the recipient status of a messages for the currently authenticates user:

```objectivec
NSDictionary *recipientStatuses = message.recipientStatusByUserID;
LYRRecipientStatus status = [recipientStatuses[layerClient.authenticatedUserID] integerValue];
NSLog(@"Recipient Status is %ld", status);
```

Applications can only mark messages as read as the system handles all other recipient states. This is done by calling `markAsRead:` on the message object.

```objectivec
NSError *error = nil;
BOOL success = [message markAsRead:&error];
```

Additionally, applications can mark a conversation as read which marks all unread messages in a conversations as read.

```objectivec
NSError *error = nil;
BOOL success = [conversation markAllMessagesAsRead:&error];
```

## Confirming Message Delivery

There a multiple ways in which Layer developers can confirm message delivery. The simplest mechansim is to visit the Layer [Logs Dashboard](/projects/layer-sample/logs) in the Layer developer portal. If the message was succesfully sent, you will see a log similar to the following:

```
May 02 2015 2:34:27pm Sync: User <USER_IDENTIFIER> created a message in conversation <CONVERSATION_IDENTIFIER>.
```

Additionally, developers can leverage both the [`LYRQueryController`](#query) object or register as an observer for the [`LYRClientObjectsDidChangeNotification`](#sync-notifications) key via `NSNotificationCenter` to be notified of incoming messages. Both are discussed in detail below.

```emphasis
**Best Practice**

Layer supports both textual and silent push on iOS. However, textual pushes are more expedient, predictable, and reliable. [Click here](https://support.layer.com/hc/en-us/articles/203174610-Can-I-send-regular-push-notifications-) to learn more.
```
