# 99cats

This project asks you to clone the (now defunct?) dress rental website
99dresses. We'll make it cat oriented.

First, follow the setup instructions in [Rails setup][rails-setup]!

We won't worry about CSRF attacks today (you're not supposed to know
what that is yet!). Take a walk on the wild side by commenting out
`protect_from_forgery` in `app/controllers/application_controller.rb`.

[rails-setup]: ../rails-setup.md

## Phase I: Cat

### Model

Build a `Cat` model. Attributes should include:

* `age`
    * Add a `numericality` validation. This will check that the user
      doesn't mistakenly upload values like "asdf".
* `birth_date`
* `color`
    * Require the user to choose from a standard set of colors.
    * Add an inclusion validation for this.
* `name`
* `sex`
    * Represent as a one-character string.
    * Add an inclusion validation so that sex is `"M"` or `"F"`.
* `description`
    * Use a `TEXT` column to store arbitrarily long text describing
      fond memories the user has of their time with the `Cat`.
* Add any necessary `presence` validations.

### Index/show pages

* Add a cats resource; generate a cats controller
* Build an `index` page of all `Cat`s.
    * Keep it simple; list the cats and link to the show pages.
* Build a `show` page for a single cat.
    * Keep it simple; just show the cat's attributes.
* Learn how to use a table (`table`, `tr`, `td`, `th` tags) to format
  the cat's vital information.

### New Form

Build a `new` form page to create a new `Cat`:

* Use text for name/age.
* Use radio buttons for sex.
* Use a drop down for color.
* Use a blank `<option>` as the default color; this will force the
  user to consciously pick one.
* You can use the `date` input type to prompt the user to pick a birth
  date. Look this up on MDN.
* Use a `textarea` tag for the description.

### Edit form

* Copy your new form to an edit view.
* You'll want to make a PATCH request, but for historical reasons
  `<form>` won't let you specify a `method` of PATCH.
    * The Rails solution is to upload a special parameter named
      `_method` with the value set to the HTTP method you want.
    * Use a hidden field to do this.
    * We say that you are "emulating" a PATCH request this way.
* Prefill the form with the `Cat`'s current details.
    * You'll use the `value` attribute a lot. You may also use the
      `checked` (for `radio`, `checkbox`) and `selected` (for
      `option`) attributes.
    * You can look up `input` and `option` attributes on MDN. It will
      explain `checked` and `selected`.
    * Note that `textarea` is not a self-closing tag. The value is the
      body of the tag.

### Unify!

* Your edit view duplicates your new view. Let's unify the two.
* Copy your edit view to a partial named `_form`.
* Change your edit view to render the partial, passing in a local
  named `cat`. Everything should still work.
* Our goal is to reuse the form for the new form too.
* To do this, we need to get three things right:
    * The edit form tries to use a `Cat`'s values to pre-fill the
      fields. The new form doesn't have an existing cat, though.
    * The edit form posts to `cat_url(cat)`; we want to post to
      `cats_url` if we're making a new cat.
    * The edit form makes a PATCH request; we want to make a POST
      reqeust.
* To solve this, build (but don't save) a blank `Cat` object in the
  `#new` action. Set this to `@cat`.
    * All the pre-filling should get the blank values.
    * Use `#persisted?` to conditionally use `cat_url(cat)`/PATCH only
      if the `Cat` has been previously saved to the DB.

## Phase II: CatRentalRequest

### Build out the model

* Make a `CatRentalRequest` model.
* Tracks `cat_id`, `start_date`, `end_date`.
* Also `status`: starts out `"PENDING"`, but can be switched to
  `"APPROVED"` or `"DENIED"`. Use a string for this.
* Add NOT NULL contraints and presence validations. Add an index on
  `cat_id`.
* Add an inclusion validation on `status`.
* Add a validation that no two **APPROVED** cat requests for the same
  cat can overlap in time.
    * To help, write a method `#overlapping_requests`, and a second,
      `#overlapping_approved_requests`, that builds on top of the
      first. **Hint:** You can use SQL's `BETWEEN` operator on date
      values.
    * Make sure not to state that a request conflicts with itself.
    * You did something similar in the polls app to prevent a user
      from answer a question twice.
* Add associations between `CatRentalRequest` and `Cat`.
* Make sure that when a `Cat` is deleted, all of its requests should
  also be deleted. Use `:dependent => :destroy`.

One last thing. When you create a new record, the `status` should be
set to an initial value of `"PENDING"`. You can use a
`after_initialize` callback to do this. Use `||=` to set `status` so
that we don't overwrite the value when we pull an approved rental
request from the DB.

### Build the controller & new view

* Create a controller; setup a resource in your routes file.
* Add a `new` request form to file requests.
* Use a dropdown to select the `Cat` desired. Your rental request form
  should upload a cat id.
* Use the `date` input type so the user may select start/end dates for
  the request.
* Add a `create` action, of course.
* Edit the cat show page to show existing requests
    * Just show the start, end dates.
    * Use `order` to sort them by `start_date`

## Phase III: Approving/denying requests

### Write `approve!` and `deny!` methods

* Add a method `#approve!` to the rental request model:
    * Move request status from PENDING to APPROVED.
    * Save the model.
    * Deny all conflicting rental requests.
* All the work of `#approve!` should occur in a single
  **[transaction][transaction-api]**.
* Most of the time, when you want to make several related updates to
  the DB, you want to do them grouped in a transaction.
* Have a chat with your TA about transactions. :-)
* Write an `#overlapping_pending_requests` to help you.
* Write a `#deny!` method; this one is easier!

[transaction-api]: http://api.rubyonrails.org/v3.2.16/classes/ActiveRecord/Transactions/ClassMethods.html

### Add buttons

* On the `Cat` show page, use `button_to` to approve or deny a cat
  request.
* You may add two member routes to `cat_rental_requests`: `approve`
  and `deny`.
* Only show these buttons if a request is pending.
* You may want to add a convenient `CatRentalRequest#pending?`
  method.
