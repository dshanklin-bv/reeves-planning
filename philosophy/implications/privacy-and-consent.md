# Privacy and Consent

*Yes, it's surveillance. Here's why that's the wrong frame.*

---

## The Honest Answer

A system that reads your messages, tracks your spending, monitors your screen time, and listens to your voice is surveillance infrastructure. Full stop. No amount of branding, mission statements, or good intentions changes the mechanical reality of what's happening at the data layer.

If you build a system that passively observes a person's life and you refuse to call it surveillance, you're either lying or you haven't thought hard enough. Reeves does the same thing your phone already does. The difference isn't in the collection. It's in what happens next.

## The Beneficiary Test

Every surveillance system answers one question: **who benefits from the observation?**

Cloud surveillance extracts value FROM the person FOR the platform. Your messages get mined so an ad network can sell your attention. Your location history gets packaged so a data broker can sell your patterns. Your screen time gets tracked so an engagement algorithm can keep you scrolling. The observer profits. The observed is the product.

Reeves extracts patterns FROM the person's data FOR the person. Your messages get analyzed so the system can remind you about commitments you made. Your spending gets tracked so it can flag when you're drifting from your budget. Your screen time gets modeled so it can notice when you're burning out before you do. The observer serves. The observed is the beneficiary.

Same data collection. Opposite value flow. This is the only distinction that matters, and it's the distinction that determines whether the system is exploitative or useful.

## Why Local-Only Changes Everything

The Mac-native thesis isn't a technical preference. It's a privacy architecture.

Data never leaves the machine. There's no cloud sync of personal observations. No API endpoint serving your behavioral patterns to a recommendation engine. No third-party access. No ad model. The person's data lives on their hardware, processed by their compute, stored in their databases.

This isn't privacy theater — a toggle in a settings menu that says "don't track me" while the data still flows to a server farm in Virginia. It's structural privacy. The data physically cannot leave because there's nowhere for it to go. No server to call home to. No telemetry pipeline. No "anonymized" usage data that's three joins away from being re-identified.

When someone asks "can my data be breached?" the answer is: only if someone breaches your laptop. Which is the same attack surface you already live with for every file on your machine. Reeves doesn't add a new vector. It operates within the one you already accepted when you bought the computer.

## What the Person Controls

The system is inspectable and governable at every layer:

**Data sources are opt-in.** Nothing connects without explicit activation. Messages, photos, finances, screen time, voice — each is a choice, not a default. The system works with whatever you give it and doesn't guilt you about what you don't.

**Observation and action are separated.** The system can know your spending patterns without having permission to move money. It can understand your calendar without being able to schedule meetings. The gap between "what it sees" and "what it can do" is a governance boundary the person defines.

**Consequential actions require human confirmation.** The governance layer gates anything with real-world impact. Send a message? Human confirms. Move money? Human confirms. The system proposes. The person decides.

**Nothing is permanently destroyed.** Soft deletes everywhere. Every observation, every inference, every model the system builds — recoverable. You can undo what the system learned. You can't undo what a cloud platform already sold.

**The knowledge graph is transparent.** You can see what the system knows about you. Not a privacy policy written in legal language designed to obscure. The actual data. The actual inferences. The actual connections. If you don't like what it concluded, you can correct it or delete it.

## What the System Must Never Do

These aren't preferences. They're architectural constraints:

- **Never send data off-device without explicit consent.** Not "implicit consent buried in a terms of service." Explicit, per-action consent with a clear description of what's being sent and where.
- **Never share personal patterns with other users or systems.** Your behavioral model is yours. It doesn't federate. It doesn't get averaged into a training set. It doesn't improve anyone's experience but yours.
- **Never make irreversible decisions without human confirmation.** The system can be fast, proactive, and opinionated. It cannot be final. Humans get the last word on anything that can't be undone.
- **Never retain data the person asked to forget.** If someone says "delete this," it's gone from the working model. The soft delete preserves recoverability, but the system stops using it. Forgetting is a right, not a feature request.

## Why This Matters for Everyone

The people most harmed by surveillance are the least powerful. The single mom in Fort Worth doesn't have a lawyer to read privacy policies. She doesn't have the technical literacy to audit what her phone sends to which servers. She doesn't have the leverage to opt out of data collection when the alternative is not having a phone.

If Reeves generalizes beyond one person, the privacy architecture must be the foundation — not a feature added in version 3 after a data breach makes the news. Local-first isn't a constraint to work around. It's the reason someone should trust this system more than the phone they already carry.

The bar isn't "better than nothing." The bar is: a person who has been burned by every platform that promised to protect their data looks at this system and says, "I believe you." That requires architecture, not promises. Structure, not policy.

The data stays on the device. The value flows to the person. The person controls what the system knows and what it can do. Everything else is negotiable. These things are not.
