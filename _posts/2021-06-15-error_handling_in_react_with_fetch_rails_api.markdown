---
layout: post
title:      "Error handling in React with fetch, Rails API"
date:       2021-06-15 15:13:11 +0000
permalink:  error_handling_in_react_with_fetch_rails_api
---


Fetch is an awesome tool that allows us to make asynchronous requests for resources. Sometimes, however, you don’t get back exactly what you were looking for. It’s in our best interest to be ready to handle these unwanted responses.

Most recently I’ve been building a single page application with a rails API backend and a react frontend. One of my resources is Comments. My fetch request for adding a new comment looks like this:

```
export function addNewComment(comment) {
  const configObj = {
    method: 'POST',
    headers: {
      "Content-Type": 'application/json',
      Accept: 'application/json'
    },
    body: JSON.stringify(comment)
  }
  return dispatch => {
    dispatch({type: "BEGIN_ADDING_COMMENT"})
    fetch("http://localhost:3001/comments", configObj)
    .then(resp => resp.json())
    .then(jsonResp => {
      if (jsonResp.status === 'error') {
        return dispatch({type: "ERROR", payload: jsonResp.message})
      }
      dispatch({type: "ADD_COMMENT", payload: jsonResp.data})
    })
    .catch(error => dispatch({ type: "ERROR", payload: error.message}))
  }
}
```

There are two different error handlers in this code. Which may seem redundant at first. If there’s already a catch for errors, why would you need more error handling?

The problem we run into is in our returned data. If our response is a valid data type, even if it contains an error from the backend, it will never trigger catch. In fact, per the MDN documentation:

> The Promise returned from fetch() won’t reject on HTTP error status even if the response is an HTTP 404 or 500.

This means we can get a bad response, and our code may still try to run normally. In this case, catch would never get called. There are a couple ways to handle this. Using my comment creation from my Rails backend as an example:

```
def create
  comment = Comment.new(comment_params)
  comment.user = User.find_by(username: params[:user])
  if comment.save
    render json: CommentSerializer.new(comment)
  else
    render json: {status: "error", message: comment.errors.full_messages[0]}
  end
end
```

Here you can see that wether a comment is successfully created or not, my response will be a json object. This is why we get a returned promise that has an error, but doesn’t trigger a failed fetch resulting in a catch.

In my example, I just checked for the error status I sent along with my error json object.

```
if (jsonResp.status === 'error') {
  return dispatch({type: "ERROR", payload: jsonResp.message})
}
```

This way, if an error is sent, I can dispatch the error accordingly, (in this case to an error reducer). And since return is used for the dispatch, the rest of my code will be skipped, preventing an erroneous object from ending up in my state.

Another option you have is using the ok object initially returned from the fetch method as a conditional. Fetch will automatically return a property called “ok” upon the promise return from fetch. This won’t trigger with a 2xx response (such as the one in my example), but it comes in handy for a 4xx or 5xx response status. per the documentation:

> …the ok property of the response (is) set to false if the response isn’t in the range 200–299.

```
fetch("http://localhost:3001/comments", configObj)
.then(resp => {
  if (!resp.ok) {
    errorHandlingFunction(resp)
  } else {
    resp.json()
  }
})
.then(jsonResp => dispatch({type: "ADD_COMMENT", payload: jsonResp.data}))
```

Note that the ok property will only be available in the first chained response from the fetch function. In the example above ok would only be available in the resp => function, not the jsonResp => function.

And of course, at the very end we can put a standard catch function to handle a promise that has failed.

Fetch responses can certainly be confusing at first. But once you understand some of the options available, it can be an awesome tool that really lets you customize what you want to do with the resources you get from it.
