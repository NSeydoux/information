Common patterns in solid apps
=============================

Even if each solid app is unique, there are patterns that ought to be recurrent in-between apps. This document aims at providing support and pointers to developers when implementing such patterns.

Let's assume that you have a webid that advertises your solid pod. If you don't, you can easily set up one on the [solid community server](https://solid.community/). For now, let us use Cleopatra's webid (https://cleopatra.solid.community) for the demonstration.

# Authentication patterns

Identity is a core component of solid, since access to resources is granted based on authentication: for a resource you control, you can explicitely grant access to others namely, or collectively, for reading, writing, and changing access rights.

## Key components

- Identity provider:
- Resource server:
- Client:

## How does access control work ?

TODO

# Read patterns

## Discover data

TODO

## Read data from a pod

# Write patterns

## Write data to a pod

Update content
application/sparql-update

## Send a notification

Notifications in solid are based on the [Linked Data Notification (LDN) specification](https://www.w3.org/TR/ldn/). Based on the following overview, introduced in the stadard, we will see how it can be introduced in a solid app. ![LDN overview](https://www.w3.org/ns/ldp/linked-data-notifications-overview.svg)

### Key components

The following terms are defined by the LDN recommandation, and we will simply here specify their meaning in the context of solid.

- Sender: The application sending the notification
- Target: The id of the entity receiving the notification
- Receiver: The pod advertised by the Target as its own, and in particular hosting its inbox.
- Consumer: The app consuming the notification from the target's inbox
- Notification: The message itself

#### Inbox discovery

Let's dereference [cleopatra's webid](https://cleopatra.solid.community/profile/card#me), with a simple HTTP GET. You should be served an RDF document in which, among other things, you will find:
```
@prefix ldp: <http://www.w3.org/ns/ldp#>.
@prefix inbox: </inbox/>.

:me
    a schem:Person, n0:Person;
    ...
    ldp:inbox inbox:;
```

From this snippet, we see that the WebId advertises for an inbox, in this case `https://cleopatra.solid.community/inbox/`, thanks to the property `ldp:inbox`. And that's it, no we can get to notification sending!

Before we carry on, please note that any resource, not only the WebId, may have an inbox. It's up to the sender and to the receiver to select the appropriate resource when sending/receiving notifications. For instance, when sending a notification specifically related to Cleopatras busy schedule, one might consider addressing it to the inbox advertised by her calendar resource, `https://cleopatra.solid.community/public/calendar/inbox/`, that you can discover by examining the response to a GET on `https://cleopatra.solid.community/public/calendar`.

#### Sending a notification

A notification is sent to the previously discovered inbox through a POST request, containing the notification body. Although it is not a strict requirement, a notification can be expressed using the ActivityStreams vocabulary (TODO: add link to voc documentation). Let's say that Caesar is having a release party for his memoirs 'De Bello Gallico', and he would like Cleopatra to attend.

First, Caesar created the event [on his pod](https://jcaesar.solid.community/public/calendar/50BC/Martius/PartyAtCaesarPalace.ttl). He then sends the invite to Cleopatra's calendar inbox:
```
POST /public/calendar/inbox/ HTTPS/1.1
Host: cleopatra.solid.community
Content-Type: text/turtle

@prefix inv: <>.
@prefix as: <https://www.w3.org/ns/activitystreams#>.
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>.
@prefix xsd: <http://www.w3.org/2001/XMLSchema#>.

inv: a as:Invite;
    rdfs:label "Invitation";
    rdfs:comment "You are invited to my book release party";
    as:object <https://jcaesar.solid.community/public/calendar/50BC/Martius/PartyAtCaesarPalace.ttl>.
```

In the response, the Location header indicates the IRI of the created event. In order to get a name nicer than a generated UUID, Caesar could have proposed a name with a Slug header in his request.
