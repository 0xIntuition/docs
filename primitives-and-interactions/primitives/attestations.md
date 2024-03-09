---
description: Attestations are cryptographically signed statements about things.
---

# Attestations

### What Are Attestations?

**Attestations** are cryptographically signed statements about a thing. When paired with Identities and Claims, Attestations become a powerful primitive for expressing how users feel about particular entities and relationships.

### Attestations In Intuition

In the Intuition ecosystem users can attest to Identities and Claims. The implication is slightly different for each. At the core of the interaction is the ability for users to state what they believe, and to place ETH behind this statement.

#### Attesting to Claims

When a user attests to a Claim, they're either _endorsing_ or _refuting_ the Claim. This involves a process of setting a user's _state respective to the Claim_ to either `1` (endorsing/affirming/agreeing) or `-1` (refuting, disagreeing). If a user takes no action (doesn't attest to the Claim), the user's state respective to the Claim is `0`. This allows for the statement that is being made (the Claim) to be abstracted away from the state of those who are attesting to agree or disagree with the Claim.&#x20;

{% hint style="info" %}
A user may not simultaneously be signaling affirmation and refutation of a Claim. Only one attestation position - in the positive or negative direction - may be active at a single time for a given Claim.
{% endhint %}

Let's look at a sample Claim:

**Bob** \[Subject] - **is** \[Predicate] - **Trustworthy** \[Object]

* This Claim describes a trait about Bob -- that he is Trustworthy.

Once this Claim has been created and becomes part of the Intuition ecosystem users are able to attest to it, either positively or negatively. Each of these actions has a different meaning.&#x20;

Once the Claim about Bob has been created, Alice and Sarah are able to attest to it. Let's look at two different situations interacting with the same example Claim.

**Situation 1**: Alice has worked with Bob and wants to attest positively (an affirmation) to the Claim _Bob is Trustworthy_ and stakes 1 ETH on her attestation.

* Alice effectively agrees with the Claim  _Bob is Trustworthy_ and believes so with the _conviction_ of 1 ETH. In this example, Alice has created a _positive attestation_ that demonstrates her agreement with the initial Claim. The amount of ETH (in this case, 1 ETH) is the level to which Alice agrees with the Claim, and this amount is her **Conviction** behind her Attestation.

**Situation 2:** Sarah has had a bad experience with Bob and wants to attest negatively (a refutation) to the Claim _Bob is Trustworthy_ and stakes .5 ETH on her attestation.

* Sarah effectively disagrees with the Claim _Bob is Trustworthy_ with the _conviction_ of .5 ETH. In this example, Sarah has created a _negative attestation_ that demonstrates her disagreement with the initial Claim. Her **Conviction** for this particular attestation is .5 ETH.

Other users can interact with the Claim in similar ways. Through viewing the direction and conviction of the attestations we can then draw our own conclusions about the Claim.  Since the statement (the Claim) and the state (user attestations) are abstracted from each other, this allows for **many-to-one attestations**. Instead of creating a new Claim for every individual attestation, there is a single Claim statement which any number of users can attest to, showing their agreement or disagreement about the statement.

#### Attesting to Identities

Attesting to Identities is more straightforward given that users can _only positively attest_ to Identities. Multiple users can still attest to a single Identity, but the expression has a slightly different meaning than attesting to a Claim. Let's look at an example of a potential lifecycle for a created Identity.

* Alice creates an Identity for AliceCo.

This Identity "AliceCo" now exists in the Intuition ecosystem, and includes the metadata that Alice initially includes, such as the name, description, and image.&#x20;

* Alice invites one of her coworkers from AliceCo to Intuition.

Alice is excited about using Intuition and wants her coworkers to explore the platform. They notice that there is an Identity for AliceCo.

* Sarah, one of Alice's coworkers, **agrees** with the AliceCo Identity and attests positively. She is in deep agreement with the initial Identity, so she stakes 1 ETH on it.

Sarah has added a _positive attestation_ to the Identity with a Conviction of 1 ETH. This can be viewed as the weight that Sarah places on her initial agreement with the Identity.

### Sharing Your Intuition

Other users that are exploring the Intuition ecosystem can see how many attestations that Identities and Claims have associated with them. This allows for users to make informed decisions about the world around them, and also allows for users to _share their Intuition_ about Identities and Claims.

