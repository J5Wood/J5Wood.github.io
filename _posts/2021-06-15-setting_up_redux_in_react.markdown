---
layout: post
title:      "Setting Up Redux In React"
date:       2021-06-15 15:09:59 +0000
permalink:  setting_up_redux_in_react
---


React is an amazing framework, but it can be a headache having to pass props down from parent to child, to child, to child, ad nauseam. Having to pass the same prop down, presumably with the same name, feels very repetitive. And god forbid something goes wrong or you have a typo, and you have to trace the prop through all your components as it’s passed all the way down to its final destination.

Enter Redux. Redux allows you to set any state for your components in a ‘store’, separate from your components, but still accessible by them. This removes the need to pass state down your component tree. It also gives you more freedom when constructing your code. Before you may have had to structure your components so that one encompassing parent held the state data for children down multiple deep branches, providing easier access to the state data by otherwise unrelated child components. With Redux, any component can be linked to the store to access your props, freeing you from the bonds of ugly forced nesting!

To get started, you will need a couple npm packages. Assuming you have already created a new react app, and you’re using npm, you will need to run :

```
npm install redux
npm install react-redux
```

Now we can set up the ‘store’ that will hold our state. This will typically be done on the top level of your application (i.e. index.js). Here you can wrap the rest of application with the redux structure as everything inside the wrapping will have access to it. Your top level component will look something like this:

```
//index.js

import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

import { createStore } from 'redux'; 
import { Provider } from 'react-redux'
import reducer from './reducer'

const store = createStore(reducer)

ReactDOM.render(
  
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

Here we are importing the createStore function. This takes in a reducer as an argument, (more on that shortly), and returns a store object that will hold our state. We are also importing a Provider component from react-redux. This is the component that holds the store, and allows everything within our app to access our store. We pass the store we created to the Provider as a store prop. The reducer we pass to createStore is where the magic of updating our state happens. We’ll come back to the reducer in a moment. Since we now have our store set up, we can first show how we get state from it and pass state back to it.

The react-redux package comes with a function used to connect our components to our state. It’s fittingly called connect, and it’s called when you export your component.

```
//extends.js

import React, { Component } from 'react'
import { connect } from 'react-redux'

class example extends Component {
  render() {
    return (
      <div>
        <h1>Example Component</h1>
      </div>
     )
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(example)
```

Connect takes two arguments. These can be named whatever you like, this is just the default standard. The order in which they’re called is important though.

The first argument, mapStateToProps, is what you will use to give your component access to the state. It is typically called this because it is used to add the state as a prop for your component. It is automatically given access to the entire state, which you pass as an argument to the function.

```
const mapStateToProps = state => {
  return {
    items: state.items
  }
}
```

Here we are giving our component a prop called `items`, and passing it the items portion of our state. Now inside our component we can call `this.props.items`, and have access to all the items from our state. We could also pass our entire state if we wanted to, or filter out a portion of our state to pass to our props, whatever state we need.

The second argument for connect, mapDispatchToProps, gives us access to a dispatch method. Dispatch is the function that takes whatever data we want to put into state, and sends it back to the store. Here we pass in dispatch as the argument, and return a function that will become part of our props.

```
const mapDispatchToProps = dispatch => {
  return {
    addToState: newData => dispatch({type: 'ADD_TO_STATE', payload: newData})
  }
}
```

There’s a few things happening here, so let’s break it down. We’re returning a function addToState that will be added to our components props. You can name this function whatever you like. When you call the function, you pass it the data you want to add to state. Now when you call `this.props.addToState(dataForState)` in your component, it will call the dispatch function. We want to pass the dispatch function an object with a key of `type:`, and another key that defines the data that we’re sending to state, here just generalized as `payload:`.

As you might imagine, we will likely have more than one component sending data to our reducer. This is what the `type:` key is for. It tells our reducer what type of action we want it to take. Here we are just calling it a very general `ADD_TO_STATE`, but you would want to name it something pertinent to the data you are adding or mutating.

With all our basic component code in place, our component would look something like this:

```
//example.js

import React, { Component } from 'react'
import { connect } from 'react-redux'

class example extends Component {

  render() {
    return (
      <div>
        <h1>Example Component</h1>
      </div>
    )
  }
}

const mapStateToProps = state => {
  return {
    items: state.items
  }
}

const mapDispatchToProps = dispatch => {
  return {
    addToState: newData => dispatch({type: 'ADD_TO_STATE', payload: newData})
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(example)
```

It is important to remember with connect, the argument order matters. The first argument will always receive state, and the second dispatch. If you wanted to use mapDispatchToProps only, you would have to set the first argument to null. Calling only mapDispatchToProps would end up passing it the state and not the dispatch function.

Now it’s time for the piece that ties it all together, the reducer. The reducer will be a function that contains our initial state, and will be responsible for handling the data we are passing into the dispatch function.

```
//reducer.js

export default function reducer(
  state = {
    items: []
  },
  action
  ) {
  
  switch (action.type) {
	
    case 'ADD_TO_STATE':
      return {
        ...state,
        items: payload
      }
			
    default:
      return state
  }
}
```

This is the reducer that we called in our createStore function in index.js. It is a function that takes in two arguments. The first being our initial state, set to whatever you need it to be. The second argument we call `action`, and it will contain all the data we sent over from our dispatch function. In our current situation it will contain `action.type`, which is currently equal to `ADD_TO_STATE`, and `action.payload`, which will be equal to whatever data we’re updating the state with. When we get to our reducer, we first hit a basic switch function, that assesses our `action.type` value. If our type is a match, we can update the state how we like with the data we’ve passed in, simply by returning a new state object. Here we’re just setting our state.items to the entire payload we’ve sent over.

You can also set a default case to handle an `action.type` that doesn’t match any of your case statements.

Note that you can, and should, name your reducer something appropriate for the specific application you’re working with, such as itemsReducer. You would just have to make sure you’re importing it and calling it under the same name in your index file as well. You can also set up your application to handle multiple reducers, making management of a larger application much more tidy. But that’s a blog for another day.

And there you have it, a state management system that’s easily accessible across your entire application. The downside to handling state in this way is it can take a fair amount of boilerplate code to get working. In a smaller application it might not make sense to go through all the trouble of setting this up. But if you’re dealing with very disconnected components that need to share data with each other, this might be a good solution for you.





