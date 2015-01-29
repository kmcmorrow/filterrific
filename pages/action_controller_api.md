---
layout: default
nav_id: controller_api
---

<div class="page-header">
  <h2>Controller API</h2>
</div>

{% include site_navigation.html %}

The Filterrific ActionController API has the following functions:

* Initialize filter settings from params, persistence or defaults.
* Execute the ActiveRecord query to load the filtered records.
* Persist the current filter settings.
* Send the ActiveRecord collection to the view for rendering.
* Reset the filter settings.



### Index action

Filterrific lives in the StudentsController's index action. Please see the code
below for notes and implementation:

```ruby
# app/controllers/students_controller.rb
class StudentsController < ApplicationController

  def index

    # Initialize filterrific with the following params:
    # * `Student` is the ActiveRecord based model class.
    # * `params[:filterrific]` are any params submitted via web request.
    #   If they are blank, filterrific will try params persisted in the session
    #   next. If those are blank, too, filterrific will use the model's default
    #   filter settings.
    # * Options:
    #     * select_options: You can store any options for `<select>` inputs in
    #       the filterrific form here. In this example, the `#options_for_...`
    #       methods return arrays that can be passed as options to `f.select`
    #     * persistence_id: optional, defaults to "<controller>#<action>" string
    #       to isolate session persistence of multiple filterrific instances.
    #       Override this to share session persisted filter params between
    #       multiple filterrific instances.
    #     * default_filter_params: optional, to override model defaults
    #     * available_filters: optional, to further restrict which filters are
    #       in this filterrific instance.
    # This method also persists the params in the session and handles resetting
    # the filterrific params.
    @filterrific = initialize_filterrific(
      Student,
      params[:filterrific],
      select_options: {
        sorted_by: Student.options_for_sorted_by,
        with_country_id: Country.options_for_select
      },
      persistence_id: 'asdf', # defaults to "controller#action" string, used for session key and saved searches
      default_filter_params: {}, # to override model defaults
      available_filters: [], # to further restrict which filters are available here
    )
    # Get an ActiveRecord::Relation for all students that match the filter settings.
    # You can paginate with will_paginate or kaminari.
    # NOTE: filterrific_find returns an ActiveRecord Relation that can be
    # chained with other scopes to further narrow down the scope of the list,
    # e.g., to apply permissions or to hard coded exclude certain types of records.
    @students = @filterrific.find.page(params[:page])

    # Respond to html for initial page load and to js for AJAX filter updates.
    respond_to do |format|
      format.html
      format.js
    end

  # Recover from invalid param sets, e.g., when a filter refers to the
  # database id of a record that doesn’t exist any more.
  # In this case we reset filterrific and discard all filter params.
  rescue ActiveRecord::RecordNotFound => e
    # There is an issue with the persisted param_set. Reset it.
    puts "Had to reset filterrific params: #{ e.message }"
    redirect_to(reset_filterrific_url(format: :html)) and return
  end

  ...

end
```

<p>
  <a href="/pages/action_view_api.html" class='btn btn-success'>Learn about the View API &rarr;</a>
</p>
