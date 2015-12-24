---
layout: post
status: publish
published: true
title: Gracefully Using Custom Primary Keys in Rails 4 Routes, Controllers, Models, Associations and Migrations
date: '2013-12-07 16:01:20'
categories:
- Code
- Ruby
- Rails
---
In Rails 4, changing the primary key of a model to something other than the default 'id' is fairly trivial. While this is technically fine, it is generally discouraged because it requires additional explicit configuration in the models and elsewhere, where Rails expects primary keys to be named 'id'. Some gems and extensions do not handle custom primary keys well, so be sure to test thoroughly when changing them.

### Migrations

First, in our ActiveRecord migrations, we can create tables in the database with a primary key other than the default 'id'...

```
class CreateParents < ActiveRecord::Migration
  def change
    create_table  :parents,
    {
      :id           => false,
      :primary_key  => :parent_name
    } do |t|
      t.string :parent_name
      #...
    end
  end
end
```

### Models
Now we need to state that primary key in our model classes...

```
class Parent < ActiveRecord::Base
  self.primary_key = "parent_name"
  #...
end
```

### Associations

Associations in models expect the foreign key field referencing a table 't' to be named 't_id', but that name is not very practical if t's primary key is something other than 'id'. We can name the foreign key field something that matches the related model's primary key...

```
class CreateChildren < ActiveRecord::Migration
  def change
    create_table :children do |t|
      #foreign-key field
      t.string :parent_name
      #...
    end
  end
end
```

...and then explicitly state the name of the foreign key field to use for the association...

```
class Child < ActiveRecord::Base
  belongs_to  :parent,
              :foreign_key => 'parent_name'</p>
  #...
end

### Routes

The _resources_ method, which generates routes corresponding to CRUD operations for a particular model, maps end-point URLs to actions in the controller. Expecting the primary key of models to be 'id', _resources_ will name the identifying parameter of a resource (taken from the matched URL) as :**id**. In the case of nested resources, each parent resource in the hierarchy is identified by a parameter with '_id' appended to the name of the resource, such as **:parent_id**, while the bottom-level resources are still identified by the **:id ** parameter...

```
Example::Application.routes.draw do
  resources :parents do
    resources :children
  end
end
```

Results in these routes:

```
  parent_children GET        /parents/:parent_id/children(.:format)             children#index
                  POST       /parents/:parent_id/children(.:format)             children#create
 new_parent_child GET        /parents/:parent_id/children/new(.:format)         children#new
edit_parent_child GET        /parents/:parent_id/children/:id/edit(.:format)    children#edit
     parent_child GET        /parents/:parent_id/children/:id(.:format)         children#show
                  PATCH      /parents/:parent_id/children/:id(.:format)         children#update
                  PUT        /parents/:parent_id/children/:id(.:format)         children#update
                  DELETE     /parents/:parent_id/children/:id(.:format)         children#destroy</p>
          parents GET        /parents(.:format)                                 parents#index
                  POST       /parents(.:format)                                 parents#create
       new_parent GET        /parents/new(.:format)                             parents#new
      edit_parent GET        /parents/:id/edit(.:format)                        parents#edit
           parent GET        /parents/:id(.:format)                             parents#show
                  PATCH      /parents/:id(.:format)                             parents#update
                  PUT        /parents/:id(.:format)                             parents#update
                  DELETE     /parents/:id(.:format)                             parents#destroy</pre></p>
```

This means our ParentController will have the param :id availble to identify the correct parent for an action, and our ChildrenController will have the param :parent_id available to it to identify the associated Parent when restricting our actions to children of a particular parent. Then we would have to retrieve our models with code like this:

```
@parent   = Parent.find(:id)
@children = Child.where(:parent_name => params[:parent_id])
```

That works, but it is misleading and suggests that the primary key of a Parent is 'id', and presumably an integer, neither of which is the case. This can cause confusion for other developers or our future-selves, especially if looking at the enummerated routes as documentation for our RESTful API. Luckily, though, Rails 4 has a very convenient way of specifying a unique name for our URL when using _resources_: the **:param** option. Passing that along to the _resource_ method will cause Rails to generate the appropriate routes.

```
Example::Application.routes.draw do
  resources :parents,
            :param => :parent_name do
    resources :children
  end
end
```

This will result in these routes:

```
  parent_children GET        /parents/:parent_name/children(.:format)           children#index
                  POST       /parents/:parent_name/children(.:format)           children#create
 new_parent_child GET        /parents/:parent_name/children/new(.:format)       children#new
edit_parent_child GET        /parents/:parent_name/children/:id/edit(.:format)  children#edit
     parent_child GET        /parents/:parent_name/children/:id(.:format)       children#show
                  PATCH      /parents/:parent_name/children/:id(.:format)       children#update
                  PUT        /parents/:parent_name/children/:id(.:format)       children#update
                  DELETE     /parents/:parent_name/children/:id(.:format)       children#destroy</p>
          parents GET        /parents(.:format)                                 parents#index
                  POST       /parents(.:format)                                 parents#create
       new_parent GET        /parents/new(.:format)                             parents#new
      edit_parent GET        /parents/:name/edit(.:format)                      parents#edit
           parent GET        /parents/:name(.:format)                           parents#show
                  PATCH      /parents/:name(.:format)                           parents#update
                  PUT        /parents/:name(.:format)                           parents#update
                  DELETE     /parents/:name(.:format)                           parents#destroy
```

Now our routes, controllers, models and associations are able to gracefully operate on models which have custom primary keys. Happy coding!
