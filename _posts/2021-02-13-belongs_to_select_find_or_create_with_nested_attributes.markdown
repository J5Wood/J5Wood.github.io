---
layout: post
title:      "Belongs_to: Select, Find, or Create with Nested Attributes "
date:       2021-02-13 00:03:33 -0500
permalink:  belongs_to_select_find_or_create_with_nested_attributes
---


When building forms for your models, you'll often want to associate the object you're creating with the object it belongs to. If you know the associated object exists already, you might use a collection_select or a find_by method to select the appropriate object. But if you want the option of creating a brand new object to associate with the primary object, using nested attributes in your form is a great option. This could cause some problems though. If you give the user the option to either select an object from a collection, OR create a new object with nested attributes, you'll have to set your controller up to appropriately handle either option. The problem comes when instantiating your object. If you select an object from the collection, the params for the parent object id will be set on your object, then the params for the nested attributes will be hit. If your controller is set up to find or create an object with those nested attributes, in its attempt to do so it will clear the id attribute it just set for the parent. Even if the nested params passed in are a blank string, it will still try to find or create an object based on that empty string. To fix this issue you just need to add a reject_if  method to your in your accepts_nested_attributes_for code in the child model. In my Rails project I made a coffee review site, and my coffee form accepted nested attributes for it's brand. This is the code from my coffee model:
```
accepts_nested_attributes_for :brand, reject_if: proc { |attributes| attributes['name'].blank? }
```

You can pass whatever validation logic you need to the reject_if block. In my case, I checked whether anything was being passed into the attribute for brand name. If there was no name present, the nested attributes would be rejected and the parent_id would remain on the child object. If the nested attributes did include a name, a new brand would be found or created based on the name passed in. And thanks to the code above both objects are instantiated by just passing your strong params to the child object when it's created or updated. Otherwise you'd have to make a workaround like this:

```
def brand_attributes=(attr) \n
  if !attr[:name].blank? \n
    self.brand = Brand.find_or_create_by(name: attr[:name], location: attr[:location]) \n
  end \n
end 
```
			
The accepts_nested_attributes_for with a reject_if statement is much cleaner, easier to implement, and eliminates the need for a conditional statement. It's a win win win!
			
			
