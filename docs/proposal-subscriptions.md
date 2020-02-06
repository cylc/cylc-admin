# Subscription Model Proposal

## The Introduction

Two of the central pillars of the new UI are incremental updates and fine-grained subscriptions.

* Incremental updates involves sending only the data which has changed to the UI, this:
    * Reduces network traffic.
    * Reduces the memory footprint of the UI.
    * Allows us to move away from using a global update loop which makes the UI more responsive.
* Fine grained subscriptions allow individual Vues (Tree, Graph, Dot, ...) to subscribe to the set o data which they actually need at a particular moment in time, this;
    * Reduces network traffic.
    * Reduces the memory footprint of the UI.
    * Reduces the number of items the UI has to process making updates faster.
    * Enables the UIServer to dynamically adjust its subscriptions to the Scheduler so as to keep load on the workflow to a minimum.

By reducing the about of data we send to the UI we reduce the load on the browser enabling us to do more with the UI and to make it more responsive.

## The Proposal

1. Individual Vues register and update their subscriptions with a service in the UI *(done)*.
2. This service "merges" subscriptions together then registers the result with the UIS *(done)*.
3. The UIS parses and stores these subscriptions.
4. When deltas arrive from the Scheduler, the UIS loops through active subscriptions to see if there is an overlap.
   * The "overlap" is the intersection of the delta and subscription fields.
   * This is event driven as we only need to do it as and when deltas arrive.
5. If there is an overlap send these fields to the UI which registered the subscription as a delta.

## The Detail

### The Three Data Stores

We have three data stores, at the Scheduler, the UIS and the UI:

* Schedulers each have their own data stores.
* The UIS has one data store containing the relevant subset of multiple Workflow Servers' data stores.
* The UI has one data store containing the requested subset of the UIS data store.

All three represent exactly the same data model. The latter two must be kept in-sync with the former.

* The UIS holds a subset of the data from the Workflow Servers.
* The UI holds a subset of the data from the UIS.

### The Ideal Subscription Interface

A subscription look something like this:

```
subscription {
  a {
    b
    c { 
      d
      e
    }
    f
  }
}
```

Deltas look something like this:

```
{
  b: 1,
}

```

```
{
  b: 1,
  c {
    d: 2
  }
}
```

A subscription is a `set`, a delta is a `dict` which is a subset of the subscription.

### The UI

#### Similar But Different Subscriptions (1)

The UI consists of different Vues which have their own subscriptions.

For example a simple Tree Vue might have the following subscription:

```
subscription {
  workflows(ids: [<id>]) {
    taskProxies {
      name
      status
      firstParent {
        id
      }
    }
    familyProxies {
      id
      name
      firstParent {
      }
    }
  }
}
```

Whereas an un-grouped Graph Vue might have this subscription:

```
subscription {
  workflows(ids: [<id>]) {
    taskProxies {
      name
      id
      status
    }
    nodesEdges {
      nodes {
        id
      }
      edges {
        source
        target
      }
    }
  }
}
```

These subscriptions are different but they have a large overlap. If we were to register both subscriptions with the UIS we would end up with two similar but different data stores in the UI. This would be in-efficient and opens up discontinuity issues.

Users may want to open many similar but different views on the same workflow so we need to be able to handle this in an elegant manner.

#### Merging Subscriptions (2)

One solution to this problem is to "merge" subscriptions together.

Subscriptions are effectively "sets" of data fields, two subscriptions can be "merged" by taking their union. For example the union of the previous two subscriptions is this:

```
subscription {
  workflows(ids: [<id>]) {
    taskProxies {
      name
      id
      status
      firstParent {
        id
      }
    }
    familyProxies {
      id
      name
      firstParent {
      }
    }
    nodesEdges {
      nodes {
        id
      }
      edges {
        source
        target
      }
    }
  }
}
```

As an added bonus we can query the UI to find out exactly what it is subscribing to and exactly which view has asked for what data making optimisation/debugging much nicer.

### The Workflow Server

When events occur the Scheduler sends "deltas" containing only the changed information.

For example if a job has changed state from submitted to running we might expect a delta of the form:

```
taskProxies {
  id: <id>,
  status: "running"
  jobs {
    id: <id>,
    status: "running"
  }
}
```

### The UI Server

#### Subscriptions (3)

When subscriptions are registered with the UIS they are stored locally in a mapping so we can associate subscriptions with open websockets.

#### Deltas (4)

When deltas arrive from the Scheduler we loop through the subscriptions and take the intersection of the delta and subscription fields, if there is an overlap, this is the delta we send to the UI.

#### Overlaps (5)

For example using the previous delta and subscription:


<table>
  <th>
    <td>Delta</td>
    <td>Subscription</td>
    <td>Intersection</td>
  </th>
  <tr>
    <td>
        Example 1
    </td>
    <td>
      <pre>
workflows(ids: [<id>]) {
  taskProxies {
    id: <id>,
    status: "running"
    jobs {
      id: <id>,
      status: "running"
    }
  }
}
      </pre>
    </td>
    <td>
      <pre>
subscription {
  workflows(ids: [<id>]) {
    taskProxies {
      name
      id
      status
      firstParent {
        id
      }
    }
    familyProxies {
      id
      name
      firstParent {
      }
    }
    nodesEdges {
      nodes {
        id
      }
      edges {
        source
        target
      }
    }
  }
}
      </pre>
    </td>
    <td>
      <pre>
workflows(ids: [<id>]) {
  taskProxies {
    id: <id>,
    status: "running"
  }
}
      </pre>
    </td>
  </tr>
</table>

## Caveats

* The comparison of delta and subscription fields will need to be fast.
  * Else the UIS will become CPU heavy and sluggish.
  * There may be potential for caching results.
* GraphQL interface does not provide a mechanism for deltas out of the box.
  * We may have to extend / implement this ourselves (can we add a `deltaSubscribe` type to GraphQL?).
  * If we did change the behaviour of the `subscribe` type 3rd party tools (incl graphiql) would loose the ability to work subscriptions (probably not a problem but worth noting).
