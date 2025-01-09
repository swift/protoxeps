# ProtoXEP: Related Entities

## What

Allow (initially MUC room-ish) entities to advertise other entities that are related to them. E.g. advertise the FDP form templates of interest to a room, or the Space to which a room belongs (and vice versa).

## How

Re-use `disco#items`, with an added `<relation/>` child element, use `node`s on the disco queries to distinguish from traditional item queries. Get an index of nodes with a `disco#items` to a defined node.

### Request a list of types of related items

If you want to know what types of related items exist on an entity, do a `disco#items` query with a node of `urn:xmpp:related:0`. A client (querying entity) may not understand all of the related item types, but potentially where many types are supported sending this query first could limit the number of type-specific queries needed, based only on the types that exist (e.g. if a client supports types 1,2,3...10 and the room only has type3 relations, the client can skip queries for the other types of relations). Result items are a set of items of jid (usually the queried entity, but possibly refering elsewhere, e.g a MUC in a Space might have its own set of FDP forms and point at a space-wide set) and an related item type identifier, stored in the node. The identifier used for a particular relation type should be defined in the specification of that type, e.g. FDP should define the type identifier for FDP forms. In the results, the node is the identifier preceded by "`urn:xmpp:related:0#`", to distinguish between disco#items queries defined in the next section being for relations and other possible uses. So if a querying entity wants to determine if relations of item type `urn:xmpp:fdp:0` exist, it should look for a node of `urn:xmpp:related:0#urn:xmpp:fdp:0`.

```xml
<iq to='witches@rooms.coven.example' type='get'>
  <query xmlns='http://jabber.org/protocol/disco#items' node='urn:xmpp:related:0'/>
</iq>
```

```xml
<iq from='witches@rooms.coven.example' type='result'>
  <query xmlns='http://jabber.org/protocol/disco#items' node='urn:xmpp:related:0'>
  <item jid='witches@rooms.coven.example' node='urn:xmpp:related:0#urn:xmpp:fdp:0'/>
  <item jid='witches-space@spaces.coven.example' node='urn:xmpp:related:0#urn:xmpp:fdp:0'/>
  <item jid='witches@rooms.coven.example' node='urn:xmpp:related:0#http://jabber.org/protocol/muc'/>
</iq>
```

### Requesting related items of a given type

Having determined that a particular related type exists from the above (or just querying directly without checking), send a `disco#items` to the entity with a node corresponding to the relation type (matches the node of an item in the query response above). These items responses are optionally extended with a child element `<relation xmlns='urn:xmpp:related:0'>` whose `rel` attribute is the relation type.

```xml
<iq to='witches@rooms.coven.example' type='get'>
  <query xmlns='http://jabber.org/protocol/disco#items' node='urn:xmpp:related:0#urn:xmpp:fdp:0'/>
</iq>
```

```xml
<iq from='witches@rooms.coven.example' type='result'>
  <query xmlns='http://jabber.org/protocol/disco#items' node='urn:xmpp:related:0#urn:xmpp:fdp:0'>
    <item jid='pubsub.coven.example' node='fdp/template/witches.example/accidentreport'>
        <relation xmlns='urn:xmpp:related:0' rel='TODO'/>
    </item>
  </query>
</iq>
```

### Relation types

A registry of relation types is maintained this document. Further relation types may be added to the document later, and changes to this list of relation types are not considered a material change to this document for e.g. purposes of considering if a change can be made to this specification once it reaches Stable or Final status. The XEP Editor may add entries to this table as required without needing further approval from another body.

This registry is currently empty.
