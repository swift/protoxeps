# ProtoXEP: XMPP Entity ACLs

## What

MUC and PubSub both have their own ACL system of restrictive affiliations. GC3 would be likely to invent a third. Instead, this aims to make a suitably generic, but suitably simple, ACL list that can be used by GC3, independent of GC3, so it can be reused elsewhere.

## How

For any given resource (e.g. GC3 channel) there will be a set of actions that can be performed on it by another entity, some standardised, and some possibly extended by particular implementations/uses - e.g. for chatrooms this might include sending messages, viewing messages, changing subject, kicking other users. Each of these actions gets a defined action identifier, e.g. `send-message`, `view-message`. A list of entity groups are defined ([XEP-0317: Hats](https://xmpp.org/extensions/xep-0317.html) use is defined here, but this is extensible), and an access of `true`, `false` or `default` can be set for each action an entity in that group may try to perform. A suitably priviledged entity can query these lists and/or modify them.

### Get list of entity groups

```xml
<iq to='witches@rooms.coven.example' type='get'>
 <acl-groups xmlns='urn:xmpp:entity-acl:0'/>
</iq>
```

```xml
<iq from='witches@rooms.coven.example' type='result'>
  <acl-groups xmlns='urn:xmpp:entity-acl:0' mutable='true'>
    <group type='urn:xmpp:hats:0' address='http://tech.example.edu/hats#TeacherAssistant' removable='false'/>
    <group type='urn:xmpp:hats:0' address='http://schemas.example.com/hats#host' removable='true'/>
    <group type='urn:xmpp:entity-acl:0' address='urn:xmpp:entity-acl:everyone:0' removable='false'/>
  </acl-groups>
</iq>
```

Note that these groups are ordered (see section on performing access checks). The presence of some groups may be enforced by deployment, or by specification (e.g. a group chat protocol might enforce that there is an 'owners' group), and these can not be removed by the querying entity (`removable='false'`). Similarly, the querying entity may not themselves have access to modify the list of groups (`mutable='false'`) either because of lack of access, or deployment constraints, or may not have access to remove a particular group (also `removable='false'`). The `removable` attribute is always present on a `group` element and the `mutable` attribute is always present on the `acl-groups` element.

### Get list of access rules for entity group

```xml
<iq to='witches@rooms.coven.example' type='get'>
  <group-access-list xmlns='urn:xmpp:entity-acl:0'>
    <group type='urn:xmpp:hats:0' address='http://tech.example.edu/hats#TeacherAssistant'/>
  </group-access-list>
</iq>
```

```xml
<iq to='witches@rooms.coven.example' type='result'>
  <group-access-list xmlns='urn:xmpp:entity-acl:0'>
    <group type='urn:xmpp:hats:0' address='http://tech.example.edu/hats#TeacherAssistant'/>
        <action id="send-message" name="Can send a message to the room" can_modify="true" value="true"/>
        <action id="kick-user" name="Can kick a user from the room" can_modify="true" value="default"/>
        <action id="destroy-room" name="Can destroy the room" can_modify="false" value="false"/>
    </group>
  </group-access-list>
</iq>
```

In this example there are three action types:

* `send-message` an entity (e.g. participant) matched by this group is allowed to perform this action (`value='true'`), and the entity (e.g. admin) fetching the list is allowed to modify this value (`can_modify="true"`).
* `kick-user`  this group doesn't define whether an entity (e.g. participant) matched by this group is allowed to perform this action (`value='default'`) so this value will be taken from a later match (see section on performing access checks), and the entity (e.g. admin) fetching the list is allowed to modify this value (`can_modify="true"`).
* `destroy-room` an entity (e.g. participant) matched by this group is not allowed to perform this action (`value='false'`), and the entity (e.g. admin) fetching the list is not allowed to modify this value (`can_modify="false"`). The reason for `can_modify` being false is determined by the use case, e.g. it might be that the service disallows any rooms being destroyed, or the querying entity may not be allowed to perform this action themselves.

### Modify access rules for entity group

Multiple accesses may be set at once within a group, but only one group is modified at once.

```xml
<iq to='witches@rooms.coven.example' type='set'>
  <group-access-list xmlns='urn:xmpp:entity-acl:0'>
    <group type='urn:xmpp:hats:0' address='http://tech.example.edu/hats#TeacherAssistant'/>
        <action id="send-message" value="false"/>
        <action id="kick-user" value="false"/>
    </group>
  </group-access-list>
</iq>
```

An empty result is returned on success.

```xml
<iq to='witches@rooms.coven.example' type='result'/>
```

### The everyone group

As the access checks are done by evaluating groups until one matches (see the section on performing resource checks), every entity needs to match at least one group that has a value defined for each action. In order to ensure this, there is always a group whose type is 'urn:xmpp:entity-acl:0' and address is 'urn:xmpp:entity-acl:everyone:0', whose ordering position must always be last, must not be removable, and must have an explit value set for every action possible on the entity.

### Add entity group

### Remove entity group

### Re-order entity groups

### Perform access check

When a resource is evaluating whether an entity is allowed to perform an action on it, it will check each entity group in turn, from first to last in the specified order, until it finds a group that matches the entity (e.g. a hats group matches when the entity has that hat) and where that group defines a value for the action other than `default`. If the value is `true` the entity is allowed to perform the action, and if the value is `false` the entity is not.

## Prior Art

Retracted [XEP-0074: Simple Access Control](https://xmpp.org/extensions/xep-0074.xml)
