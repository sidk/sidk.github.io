---
id: 390
title: How To Keep Your Controllers Thin with Form Objects
date: 2017-05-26T17:13:36-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/338-autosave-v1/
permalink: /338-autosave-v1/
---
Typically in Rails, forms which post data to a `#create` or `#update` action are concerned with one particular resource. For example, a form to update a user&#8217;s profile. The sequence looks something like this:

  1. The form submits some data to the `#create` action in `UsersController`.
  2. The `#create` action, with something like `User.create(params)` creates the record in the `User` table. 

This is easy to reason about. We have one form to create one user. It submits user data to a users controller, which then creates a record in the User table with a call to the `User` model.

Though some percentage of the forms in our app can be described this simply, **most forms we find ourselves needing to build are not that simple**. They might need to save multiple records, or update more than one table, and/or perform some additional processing (like sending an email). There might even be more than one context under which a record can be updated (for example a multi-step application form).

Trying to stuff a bunch of logic in your controller can turn out to be quite painful in the long run, as they become longer and harder to understand. Moreover, you might find yourself having to perform gymnastics with your views to get them to pass in parameters correctly.

For example, imagine you&#8217;re building a form for a project management app. This form creates a user like in our first example, but in addition, has to do a few other things. First, it has to create a record in the `Message` table. Then it has to create a record in the Task table. Finally it has to send an email to the project manager.

Some definitions and assumptions before we go forward:

  1. The `Message` table is used for internal messaging amongst various members of the project.
  2. The `Task` table is used to store and manage tasks assigned to members of the project.
  3. When a new user is created, we want to send an internal message and assign a new task to the project manager. 
  4. We also want to send an email notification to the project manager when a new user is created.

Compared to the first example, it&#8217;s obvious that this form requires a flow that doesn&#8217;t quite as neatly fit in to the conventional form flow that we saw earlier. There are a few ways we can handle this:

  1. Add in the code to `UsersController` to accomplish the extra stuff.
  2. If your models are associated with each other (via `has_many` or `belongs_to`), build a [nested form](http://guides.rubyonrails.org/form_helpers.html#nested-forms) using either Rails&#8217; built-in `fields_for` helper, or Ryan Bates&#8217; [nested_form](https://github.com/ryanb/nested_form) gem. You&#8217;ll still have to send the email in the controller.
  3. Create a new controller and corresponding &#8220;form object&#8221; that encapsulates what you want to do. 

### What is a Form Object

A form object is a Plain Old Ruby Object (PORO). It takes over from the controller wherever it needs to talk to the database and other parts of your app like mailers, service objects and so on. A form object typically functions together with a dedicated controller and a view.

### Why use a Form Object

Now, while there are good reasons to go with either option 1 or 2, I&#8217;m going to elaborate on option 3. There are a few benefits to using a form object in a situation like this:

  1. Your app will be easy to change.
    
    If your app is a decent sized web-app, you will likely have a multitude of paths and views through which data gets saved and/or retrieved from your Users table. It&#8217;s worth thinking about if `UsersController` or the `User` model should be where these paths meet, because if you&#8217;re not careful your controller can devolve into a mess of hard-to-change conditional code.

  2. Your app will be easy to reason about
    
    Conventionally in Rails, the simplest way to reason about a controller is to have it concerned with the seven RESTful actions and views; these actions would only interact with one model and ideally the interaction would be a one-liner like `@user.update(params)`. The closer your controller is to this pattern, the easier your controller will be to reason about.

  3. Your view will likely be simpler to write (read: no deeply nested forms) and contain less logic.

### What a Form Object looks like in practice

**First things first, decide on the name of your new controller.** This will give you quick feedback on if your abstractions will make things easier to reason about.

I always ask myself this, what happens when the form is submitted? In our example above, a user is created and the rest of the project team is notified. So a reasonable name might be &#8216;UserRegistrationsController\`. You can double check by trying to apply the seven RESTful actions to this controller:

  * Can I &#8220;create&#8221; a user_registration?
  * Can I &#8220;update&#8221; a user_registration?
  * Can I &#8220;destroy&#8221; a user_registration? 
  * &#8230; and so on

All seven actions might not always make sense, which is why I find it handy to define my routes like this, for example:

    resource :user_registrations, only: [:create, :update, :new, :edit] 
    

In my Rails apps, the convention I follow is to always name the form object class `FormObject`, and namespace them with something context dependent. So in this case, my form object would be `UserRegistration::FormObject`.

So your form, which resides at `user_registrations/new.html.erb`, posts some data to the `#create` action of the `UserRegistrationsController`, which calls `#save` on `UserRegistration::FormObject` with the params you pass in.

<div id="mc_embed_signup" style="background: #dff7fe; padding: 15px;">
</div>

### Form Object parameters

To be able to specify an input in your form, you need to expose the related attribute in your form object. In our example, one of the inputs we want is the user&#8217;s name. So in our form object, we&#8217;d do:

    module UserRegistration
      class FormObject
        include ActiveModel::Model
        attr_accessor :name
        ...
        ...
        def self.model_name
          ActiveModel::Name.new(self, nil, 'UserRegistration')
        end
      end
    end
    

Because we&#8217;ve said `attr_accessor :name`, we can now in our view say something like `<%= f.text_field :name %>`.

You&#8217;ll also notice a couple of other things:

  1. We said `include ActiveModel::Model`. This is cool, because we can now use methods like `validates` and perform any validations we want. An advantage of using validations in the form object is that they are specific to the form object and won&#8217;t clutter up your model(s).

  2. We also defined the `model_name` class method. The Rails form builder methods (`form_for` and the rest) need this to be defined.

### Form Object Initialize and Save

The behavior of our form object will be governed by two methods. `#initialize` and `#save`. This is because in our controller, we want to be able define our create action like so:

    def create
      @form_object = UserRegistration::FormObject.new(params)
      if @form_object.save
        ... #typically we flash and redirect here
      else
        render :new
      end
    end
    

You can see above how the form object directly replaces the model, allowing us to keep our controller clean and short.

Your initialize method would look something like:

    def initialize(params)
      self.name = params.fetch(:name, '')
      ... # and so on and so forth
    end
    

And save:

    ...
    def save
      return false unless valid? #valid? comes from ActiveModel::Model
      User.create(name: name)
      notify_project_manager
      assign_task_to_project_manager
      true
    end
    
    private
    
    def notify_project_manager
      ... # here we talk to the Message model
    end
    
    def assign_task_to_project_manager
      ... # here we talk to the Task model
    end
    ...
    # and so on and so forth
    

The important thing with the `#save` method is to return false and true correctly depending on if the form submission meets your criteria, so that your controller can make the right decision.

### Recap

I&#8217;m a big fan of form objects, and a general rule of thumb for me is to use them whenever I feel things are getting complicated in the controller and view. They can even be used in situations where you&#8217;re not dealing directly with the database (like for example interacting with a third party API).

I encourage you, if you haven&#8217;t already, to consider how form objects might fit into your app. They will go a long way in ensuring your controllers _and_ models are thin and keeping your code maintainable.

Have you used form objects in your Rails apps? Have they helped or hindered you? How else do you keep your controllers thin? Let me know in the comments section, I&#8217;d love to hear what you think.

<div id="mc_embed_signup" style="background: #dff7fe; padding: 15px;">
</div>