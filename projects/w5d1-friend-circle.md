# Social Thingamajig

## Phase I: `User`, `UsersController` and `SessionsController`

* Sign up (`email` instead of `username`), log in, log out. Go.  (Use
  *[BCrypt][bcrypt], please - we want our fake users to have some
  *peace of mind)*

[bcrypt]: https://github.com/codahale/bcrypt-ruby

## Phase II: `Circle`

###NB: Be careful not to name your Application and Model the same thing.


* Allow users to create a `Circle` to share with.
* Allow them to select other `User`s to put in the circle.
    * To start, present check-boxes.
    * Upload an array of `*_ids`.
    * Make sure each of your form inputs has a unique `id` attribute.
    * You can ensure this by embedding the `user.id` in the attribute
      name.
* You'll probably want a join table, `circle_memberships`. A
  user can be part of many circles and a circle has many members.
* Remember to `permit` your `*_ids` attribute in the controller,
  since the `CirclesController` could mass-assign it on
  `Circle` creation. Take a look at the "Methods added by
  has_many" section in the Rails Guide on associations.

## Phase III: `Post` and `Link`s
[Nested form time! Yay!][Nested form time!]
[Nested form time!]: https://github.com/appacademy/rails-curriculum/blob/master/w5d2/transaction.md

Users will be making posts which they will share with friend circles
that they select. Each post will have many links, and we'll be
creating the post and the links all in one form.

* Allow `User`s to make a `Post`.
* They can select multiple of their `Circle`s to share the new
  post with (checkboxes).
    * You'll probably want a join table, `post_shares`, to connect a
      `Post` with multiple `Circle`s.
    * Again, ensure each checkbox input tag has a unique HTML id.
* Each `Post` has a body, which is a description of the collection of
  links.
* Each `Post` also has a collection of associated `Link`s. `Link`
  should be its own model class.
* A user will POST the post data and the link data together to the
  same controller action. Ensure the links data and the post data are
  named well (`params[:post]` & `params[:links]`) so that you can access
  them separately in your create action.
* Allow the user to submit up to three `Link`s to start (i.e. display
  3 nested forms for links).
    * The user may not want to upload as many as three links so make
      sure you don't create empty link objects.

[nested-form]: https://github.com/appacademy/rails-curriculum/blob/master/w5d2/transaction.md

## Phase IIIB: Editing `Circle`

Use your old form so that you can edit a `Circle`. Factor it out
into a partial for reuse. Allow the user to return and re-edit the
circle.

Obviously, embed the current attributes in the edit form. Don't make
the user type in all the attributes again.

The user may wish to remove members of the `Circle`. What about
if you uncheck all the `User`s from the `Circle`? Try it.

The problem is that if no users are selected, then the form has
nothing to upload for `user_ids`. Instead of uploading an empty array
for `user_ids`, it just doesn't upload anything at all. That means
that in the `update` action, if we call
`Circle#update_attributes`, `user_ids` won't appear among the
attributes to update, so it will remain unchanged.

The solution is to add a "bumper" hidden input:

    <input type="hidden" name="circle[user_ids][]" value="">

This way the form will always upload **at least** one value for
`circle[user_ids]`: `""`. `user_ids=` will ignore the `""`
value: `user_ids = [1, 2, ""]` will add memberships for users \#1 and
\#2. `user_ids = [""]` will clear all memberships.

You need to use this trick whenever you have checkboxes that you want
to be able to uncheck to indicate a deletion.

## Phase IV: Feed

A user should be able to see all the posts shared with them.

Use the `PostShare` model you created earlier.

* Create a `/feed` route
* It should display the items shared with the current user
  (posts with bodies and associated links, along with the
  author of that post)

## Later Functionality

* Upvoting
* [Pagination][kaminari]
* Comments
* Password reset

[kaminari]: https://github.com/amatsuda/kaminari

