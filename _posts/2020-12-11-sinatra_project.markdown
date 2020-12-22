---
layout: post
title:      "Sinatra: Protecting Your Dynamic Routes"
date:       2020-12-11 23:34:20 -0500
permalink:  sinatra_project
---


   When building a web app it's important to protect your data. One way we do this is to limit the data that gets passed to and from our view files. Unfortunately we need to pass some data around due to HTTP being stateless. This means we are bound to expose some information to the user. With forms we can give users a chance to change the dynamic route that the form is being sent back to.
	
  I created a web app to create and store motorcycles in a personal garage. When I create a form to patch my object, it leaves this html exposed to the user:
	
```
<form method="POST" action="/motorcycles/1">
  <input type="hidden" name="_method" value="patch">
  ...
```


  If you were to change the 1 to a number that matched the ID of another motorcycle, and submitted the form, you could very well end up changing the attributes of another motorcycle object. I found two ways of preventing this for my app. 

  One way was by authenticating the user had the authority to edit the object sent  to the patch request. I enabled sessions in my app and when logging into my site a session key of [:user_id] is set to the users ID. This keeps track of what user is currently signed in, which will allow us to verify a users identity. At the beginning of the patch request I found the motorcycle object requested by the params{;id] key. 

```
patch '/motorcycles/:id' do
  motorcycle = Motorcycle.find_by_id(params[:id])
  redirect_if_not_owner(motorcycle)
  ...
```

Then, in a class helper method, I checked to see if the motorcycle object belonged to the currently signed in user based on the current session[:user_id].

```
def redirect_if_not_owner(motorcycle)
  if motorcycle.user.id != session[:user_id]
    flash[:message] = "You may Only Edit Your Own Motorcycle"
    redirect '/motorcycles'
  end
end
```
		
  If the post methods route is changed to a motorcycle that doesn't belong to the user and then the form is sent, the motorcycle.user.id will not equal the session[:user_id] and the form will redirect to another route.
		
  This works because of the objects relationships defined in ActiveRecord. Since the motorcycle 'belongs_to' a user you can check it's user ID. However what if you were dealing with an object that doesn't belong to you? The object wouldn't have an ID for you to compare with your session[:user_id]. I ran into this issue when editing motorcycle brands on my site. For this I created a new key => value pair in the session hash when a get request was sent to a specific brands edit page. 
		
```
get '/brands/:id/edit' do
  redirect_if_not_logged_in
  @brand = Brand.find_by_id(params[:id])
  redirect_if_bad_route(@brand)

  ###  set session[:brand_id] to the brand.id  ###
  session[:brand_id] = @brand.id

  erb :'brands/edit'
end
```
		
  Then when the patch request is received,I find the brand based on the params passed in from the form. Then I call a method to redirect in case the ID of the brand just passed in differs from the original edit get request.
		
```
patch '/brands/:id' do
  brand = Brand.find_by_id(params[:id])
  redirect_for_wrong_brand(brand)
  ...
```
		
```
def redirect_for_wrong_brand(brand)
  if brand.id != session[:brand_id]
    flash[:message] = "Invalid Brand Change"
    redirect "/brands"
  end
end
```
			
  If the params[:id] key changed on the form submission, a different brand will be found in the patch route, and it's ID will not match the ID stored in the session key. This ensures the object initially requested for editing is the same as the object that's being patched, and keeps the ID that's being used for confirmation out of the users reach. 
			
  These are both small but important steps you can take to keep users from accessing and editing your data.
			
Link to app repository: (https://github.com/J5Wood/motorcycle-garage)
	
		
		
		
		
