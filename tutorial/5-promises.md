---
layout: default
title: Working with Promises
description: Asynchronous flow control and flux state
permalink: /tutorial/promises
categories: [tutorial]
---

# Dispatching Promises

Not all actions are synchronous. When working with real world applications, it's frequent to see Actions that use 
Promises to fetch data from a remote server using either [Fetch API](https://github.com/github/fetch) or a polyfill. 
In this case it's important to be able to handle three possible states: loading, successful response, and error.

Any action that returns a Promise will be handled as asynchronous Actions:

`actions/AsyncLocationActions`

```js
export default alt.createActions('LocationActions', {

  fetchLocations() {
    // your fetch request would go here.
    // We will just mock something for simplicity
    return Promise.resolve([ 'Madrid', 'Berlin', 'San Francisco' ]);
  }

});
```

We can bind the Store to these Actions as usual, and Promises will be resolved before dispatching the value to the Store. 

# Dispatching extra states: loading and error

You have full flexibility about how to dispatch the `loading` and `error` state, for example using the standard attributes of a [Flux Standard Action](https://github.com/acdlite/flux-standard-action). We can add this to the previous example:

```js
export default alt.createActions('LocationActions', {

  fetchLocations() {
    // notify the loading state immediately
    this.dispatch({
      meta: { loading: true }
    });
    return Promise.resolve([ 'Madrid', 'Berlin', 'San Francisco' ]).catch((errorMessage) => {
      this.dispatch({ 
        error: true, 
        meta: { errorMessage } 
      });
      throw error;
    });
  }

});
```


```js
import asyncLocationActions from '../actions/AsyncLocationActions';

class LocationStore extends Store {

  constructor() {
    this.state = {
      locations: [],
      loading: false,
      error: undefined
    }

    this.bindAction(asyncLocationActions.fetchLocations, this.onFetchData)
  }

  onFetchData(locations, { error, meta: { loading, errorMessage }}) {
    this.setState(locations, error: errorMessage, loading);
  }

}
```

The second argument is an instance of [Standard Flux Action](https://github.com/acdlite/flux-standard-action), 
including an `error` and `loading` attributes.  

The action handler will be notified first when the Promise gets instantiated with `loading=true`. 
The next update will either include either a successful response or an error message.

The view must change slightly to display an error message if it exists or a spinner if the content is loading.

```js
class LocationsComponent extends React.Component {

  componentDidMount() {
    // initial fetch, will trigger two re-renders: 
    // "loading" and "results received"
    asyncLocationActions.fetchLocations();
  }

  render() {
    const { locations, loading, error } = this.props;
    if (error) {
      return <div class="error">{ error.message }</div>;
    }

    if (loading) {
      return <div class="spinner">Loading...</div>;
    }

    return (
      <ul>
        {
          props.locations.map(function(location, index) => {
            return <li key={index}>{location.name}</li>
          })
        }
      </ul>
    );
  }  

}

function LocationsWrapper() {
  return (
    <AltContainer store={locationStore}>
      <LocationsComponent />
    </AltContainer>
  )
}

```

That's it: define the Actions using Promises, receive the state in the Store, trigger an update of the View. 

You have completed the tutorial. Now, to know more about alt you can either go to [the documentation](../doc) 
or take a look at [the test suite](https://github.com/koliseoapi/alt/tree/master/test).