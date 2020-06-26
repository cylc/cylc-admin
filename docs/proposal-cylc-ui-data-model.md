# Cylc UI Data Model

See also the closely related [subscription model](proposal-subscriptions.md)

Views state the data they require to function in something akin to a manifest:
```
taskProxies = [
    'id',
    'name',
    'status',
    'isHeld'
],
jobs = [
    'id',
    'status',
    'submitNum'
]
```

The workflow component (which is the mount point for the "cylc views") takes
this manifest and issues a subscription on behalf of the view. Ideally this
would involve subscription merging but if you separate the data from the query
with a nice shiny interface what goes on under the hood doesn't really matter.

Done!

At the moment this all happens via the gquery (bad pun on graphql / jquery)
file which I put in back in the early days. Each view is registered and can
issue multiple subscriptions. But the views themselves never handle
subscriptions, or manage them in any way. The subscriptions are created and
destroyed automatically. It was intended to be temporary but I had expected the
register/subscribe interface implemented there to be evolved and carried
forward, albeit in a wildly different form.

The more involved things are:

- Trees/Deltas need to be available on request, there are two logical options here, centralised or decentralised, both should be relatively efficient. I've told Bruno I'm happy to swing either way. I think he's got a way forward there, I'm pretty happy with whatever so long as it's efficient.
  I think this manifest should be mutable since views have state and might want different things at different times. For example with the table view we only need to request the data for the table columns the user has chosen, not the full number of available columns. In Vue speak this would be a computed item i.e:

```
export default {
   // ...
   computed: {
     maifest () => {
       if (this.isExpanded) {
         return this.expandedFields
        } else {
          return this.collapsedFields
        }
     }
   }
```

I said this happens in the workflow component, but at the moment it actually
happens in the workflow service which is how we are able to persist
subscriptions past the lifecycle of the workflow component itself. Thanks to
vuex it's pretty easy to store stuff centrally.

GScan, the Dashboard currently go via this service too, everything gets merged
into a single subscription under-the-hood.

The idea is that we have a centralised service which handles the requirements
of the live components and shields them from all the pesky implementation
details like websockets, graphql, query merging, etc.

The danger is moving the subscriptions into the components themselves, it
allows for rapid development yes, but it allows you to make assumptions which
aren't reliable, Like for instance when you control the subscription you
control the filtering. But with a central solution you don't get that. Once
you've gone distributed it could be a nasty piece of work to re-centralise
things and find ways around the subscription-orientated model.
