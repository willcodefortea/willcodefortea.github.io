---
title: "Chain of Responsibility"
categories: what-is
layout: single
tags:
  Design Pattern

header:
  overlay_image: /assets/images/what-is/chain-of-responsibility/chain-banner.png
  overlay_filter: 0.6
excerpt: The Chain of Responsibility is a behavioral design pattern that passes a request along a set of handlers, each one deciding to process the request or pass it to the next handler.
---

## The Pattern

The Chain of Responsibility encapsulates a behaviour into an object usually referred to as a handler. Each handler then stores a reference to the next handler in the chain, and can decide if it should process that request or pass it to the next handler in the chain.

> The Chain of Responsibility is essentially an Object Orientated version of if ... else if... else if ...

There are some considerations around the execution of a chain that are worth remembering.

1. A chain can be composed of a single item
2. A chain may not result in the handling of a request at all
3. Requests may not get to the end of the chain

### When to use

This pattern is best used when there are multiple objects that could potentially handle a request, and when that decision is made at runtime.

## Example

Let's consider an automated system for raising incidents in an e-commerce platform. Incidents have two properties; a type and a severity. Both of these things are defined by your business and both will impact what has to be done in response. These may be things like:

1. Notify senior leadership
2. Update the status page
3. Stop users from buying anything (extreme, but possible)
4. ... many others that are likely to grow in time

In code, this might begin by looking something like this:

```typescript
type IncidentType = "Security" | "Network" | "Data Breach";

type Severity = "SEV 0" | "SEV 1" | "SEV 2"

const handleIncident = (incidentType: IncidentType, sev: Severity) => {
  if (type === "Security") {
    if (sev === "SEV 0") {
      notifySeniorLeadershipTeam();
      notifySecurityEngineers();
    } else if (sev === "SEV 1") {
      notifySecurityEngineers();
    }
    notifyOnCallEngineer();
  }
  else if (type === "Network") {
    if (sev === "SEV 0") {
      notifyOnCallEngineer();
    } else {
      waitAndNotify()
    }
  }
  else if (type === "Data Breach") {
    notifySeniorLeadershipTeam();
  }
}
```

We're only dealing with a couple of use cases but already things are getting out of hand. It's difficult to see when and who are notified, and this problem will only get worse as the number of types or response behaviours increases. Let's try out our Chain of Responsibility.

### Step 1 - Define the handler interface

```typescript
interface Request {
  incidentType: IncidentType;
  sev: Severity;
}

class BaseHandler {
  next: BaseHandler | undefined;

  constructor() {
    this.next = undefined;
  }

  handle(request: Request) {
    // reduce boilerplate by handling common behaviour in the parent
    if (this.next) {
      this.next.handle(request);
    }
  }
}
```

### Step 2 - Create concrete classes

Each handler makes two decisions:

* if it'll process the request
* if it'll pass the request to next handler along the chain

```typescript
class SecurityHandler extends BaseHandler {
  handle(request: Request) {
    if (request.incidentType != "Security") {
      super.handle(request);
      return;
    }

    if (sev === "SEV 0") {
      notifySeniorLeadershipTeam();
      notifySecurityEngineers();
    } else if (sev === "SEV 1") {
      notifySecurityEngineers();
    }
    notifyOnCallEngineer();
  }
}

class NetworkHandler extends BaseHandler {
  handle(request: Request) {
    if (request.incidentType != "Network") {
      super.handle(request);
      return;
    }

    if (sev === "SEV 0") {
      notifyOnCallEngineer();
    } else {
      waitAndNotify()
    }
  }
}

class DataBreachHandler extends BaseHandler {
  handle(request: Request) {
    if (request.incidentType != "Data Breach") {
      super.handle(request);
      return;
    }

    notifySeniorLeadershipTeam();
  }
}
```

### Step 3 - Build the chain

Depending on the problem, clients may build chains dynamically or use prebuilt ones. Our example calls for a prebuilt chain, but if a dynamic one is required then a factory can be used to handle the creation for you.

```typescript
const buildChain = (handlers: BaseHandler[]) => {
  let baseHandler = handlers.shift();
  let prev = baseHandler;

  while (handlers.length > 0) {
    const current = handlers.shift();
    prev.next = current;
    prev = current;
  }

  return baseHandler;
}

// The ordering here will be the order in which the handlers are invoked
const incidentHandlers: BaseHandler[] = [
  new SecurityHandler(),
  new NetworkHandler(),
  new DataBreachHandler(),
]
export const incidentChain = buildChain(incidentHandlers);
```

{% include figure image_path="/assets/images/what-is/chain-of-responsibility/chain-of-responsibility.jpg" alt="Flow chart of a chain of responsibility." caption="Each handler in the chain can decide to handle the request, or pass it to the next handler in the chain." %}{: .img-zoom}


### Step 4 - Replace the code

Now we have our chain, all that remains is to replace our old code with it.

```typescript
type IncidentType = "Security" | "Network" | "Data Breach";

type Severity = "SEV 0" | "SEV 1" | "SEV 2"

const handleIncident = (incidentType: IncidentType, sev: Severity) => {
  incidentChain.handle({ incidentType, sev });
}
```

The are many benefits to the application of this pattern in this case. We've create more cohesive code that adheres to the [Single Responsibility](https://www.willcodefortea.com/what-is/solid-principles-s/) and [Open / Closed](https://www.willcodefortea.com/what-is/solid-principles-o/) design principles, resulting in code that is much easier to understand and test. \o/
