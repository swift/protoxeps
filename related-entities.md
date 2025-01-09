# ProtoXEP: Related Entities

## What

Allow (initially MUC room-ish) entities to advertise other entities that are related to them. E.g. advertise the FDP form templates of interest to a room, or the Space to which a room belongs (and vice versa).

## How

Re-use `disco#items`, with an added `<relation/>` child element, use `node`s on the disco queries to distinguish from traditional item queries. Get an index of nodes with a `disco#info` to a defined node.

### Request a list of types of related items

If you want to know what types of related items exist on an entity, do a `disco#info` query with a node of `urn:xmpp:related:0`. A client (querying entity) may not understand all of the related item types, but potentially where many types are supported sending this query first could limit the number of type-specific queries needed, based only on the types that exist (e.g. if a client supports types 1,2,3...10 and the room only has type3 relations, the client can skip queries for the other types of relations). Each node is represented as a feature in the result. The identifier used for a particular relation type should be defined in the specification of that type, e.g. FDP should define the type identifier for FDP forms.

```xml
<iq to='witches@rooms.coven.example' type='get'>
  <query xmlns='http://jabber.org/protocol/disco#info' node='urn:xmpp:related:0'/>
</iq>
```

```xml
<iq from='witches@rooms.coven.example' type='result'>
  <query xmlns='http://jabber.org/protocol/disco#info' node='urn:xmpp:related:0'>
  <feature var='urn:xmpp:fdp:0'/>
  <feature var='http://jabber.org/protocol/muc'/>
</iq>
```

TODO: I think this is probably not a good re-use of #info, and we should define an additional protocol here. Thoughts?

### Requesting related items of a given type

Having determined that a particular related type exists from the above (or just querying directly without checking), send a `disco#items` to the entity with a node corresponding to the relation type (matches the feature in the `#info`). These items responses are optionally extended with a child element `<relation xmlns='urn:xmpp:related:0'>` whose `rel` attribute is the relation type.

```xml
<iq to='witches@rooms.coven.example' type='get'>
  <query xmlns='http://jabber.org/protocol/disco#items' node='urn:xmpp:fdp:0'/>
</iq>
```

```xml
<iq from='witches@rooms.coven.example' type='result'>
  <query xmlns='http://jabber.org/protocol/disco#items' node='urn:xmpp:fdp:0'>
    <item jid='pubsub.coven.example' node='fdp/template/witches.example/accidentreport'>
        <relation xmlns='urn:xmpp:related:0' rel='TODO'/>
    </item>
  </query>
</iq>
```

### Relation types

Some core relation types are defined here, further types may be defined in the specifications defining the related item type. (e.g. FDPs might have an allowable rel of 'template' or 'submitted', potentially).

TODO: What can go in the `rel` attribute? Matthew?
