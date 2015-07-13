#Metadata
Metadata provides an elegant mechanism for expressing and synchronizing contextual information about Conversations. This is implemented as a free-form structure of string key-value pairs that is automatically synchronized among the participants in a conversation. Example use cases of metadata include:

1. Setting a conversation title.
2. Storing information about participants within the Conversation.
3. Attaching dates or tags to the Conversation.
4. Storing a reference to a background image URL for the Conversation.

The following demonstrates setting `metadata` on a conversation:

```java
Map<String, Object> metadata = new HashMap<String, Object>();
metadata.put("title", "My Conversation");

Map<String, Object> theme = new HashMap<String, Object>();
theme.put("background_color", "333333");
theme.put("text_color", "F8F8EC");
theme.put("link_color", "21AAE1");
metadata.put("theme", theme);

metadata.put("created_at", "Dec, 01, 2014");
metadata.put("img_url", "/path/to/img/url");

//Assign the metadata to the conversation.
//Pass in 'true' to merge the HashMaps (all old keys will be preserved and new values will be set)
// or 'false' to replace the existing HashMap (all old key/pairs will be lost)
mConversation.putMetadata(metadata, false);
```

For convenience and to facilitate the namespacing of information within metadata, values may be manipulated as key paths. A key path is a dot (.) delimited string that identifies a series of nested keys leading to a leaf value. For example, given the above metadata structure, an application could change the name of a participant via the following:

```java
mConversation.putMetadataAtKeyPath("theme.background_color", "FFFFFF");
```

Applications can fetch metadata for a given conversation by accessing the public `metadata` property on `Conversation` objects.

```java
Map<String, Object> current = mConversation.getMetadata();
mTitleTextView.setText((String)current.get("title"));
```

While this feature is both powerful and convenient, it is not designed to serve as a general purpose data store. It's important to understand the intended use cases and limitations associated with the feature to ensure an ideal user experience. Please see the [Metadata Guide](/docs/android/guides#metadata) for more detail.
