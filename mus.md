# ProtoXEP: Message Unread Synchronisation

## What

[XEP-0490: Message Displayed Synchronization](https://xmpp.org/extensions/xep-0490.html) allows synchronising details of the last displayed/read message. This allows you do e.g. show a bar in a chat beyond which messages are 'new', or a UI to jump back to the first unread message. It doesn't help with knowing at the start of a session, without querying the archive, if there are new messages in a chat later than this last displayed message. So this adds that.

## How

Add a new iq for setting MDS, instead of PEP publishes, server tracks which chats have an unread marked. When receiving messages, it flags them as unread, and the latest known ID. When client marks read, it sends the ID. Server updates the 490 list, and if the ID matches the latest unread one, it clears the unread flag.

On login client sends an iq (or uses bind2) to enable MUS. It gets an iq result with current unread list, and a push when the unread state is modified.

Also interacts with [XEP-0437: Room Activity Indicators](https://xmpp.org/extensions/xep-0437.html) to allow setting an unread marker without receiving the messages involved. How this then interacts with the client providing the read-id needs solving.

Server advertises a feature that it supports this, which means the client then doesn't need to do 490 publishes manually.
