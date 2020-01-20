---
layout: post
title:      "Devising a Template-Driven Methodology for a Ruby On Rails App."
date:       2020-01-20 09:55:01 -0500
permalink:  devising_a_template-driven_methodology_for_a_ruby_on_rails_app
---


For my Ruby-On-Rails project I wanted to create a rails app that mimicked a robust content/project management system similar to Trello or Basecamp.  Taking cues from office colleagues in the home-building industry, I decided to structure the app  –  “Doc-Helper” – ostensibly as a tool to manage the flow of documents in an accounting business that handles multiple home-builder clients.  

Users are defined to have `roles` either as a  `customer`, `manager`, or `administrator`.  Documents, called `Packages` in the app, are created as nested elements in customizable `Accounts`, which are then nested inside defined `Company` objects.  Users, depending on their `role`, are given access to one or more `companies`, allowing them to create new `accounts`, `packages`, and create/delete `package_comments`.

Users are able to navigate through the app to view all elements the `companies` they belong to, as well as bring up all other `users` (aliased as `associates`) who also have access to those companies.

I wanted the app to maintain a consistent dashboard-style UI throughout all its views.  Forms for creating or editing elements in the app also needed to maintain consistency.  For simple RESTful architecture, basic view files pointing to reusable partials, with a handful of helper methods supporting the background, seemed to suffice at first.  However, when I began experimenting with views taken from different frames of reference, things quickly began to become complicated and unmanageable.  For Example:

Take the following URLs in the app:

```
http://localhost:3000/users/9/
```
points to the current_user's homepage.  The user’s homepage is a content-rich dashboard of all related content in the current_user's profile, including listed content about their associated companies, accounts, etc...

```
http://localhost:3000/users/9/companies
```
Becomes (at least at this point in the app’s development) redundant: all data `current_user` would find in the `company#index` is essentially available from their homepage.

```
http://localhost:3000/users/9/companies/7
```
However, is useful.  Although it still more-or-less shows the same degree of information as what is found on the homepage, it filters in only the `company/7` content.

```
http://localhost:3000/users/9/associates?role=contact
```
is used as a filtering route for accessing the different associates attached to the companies `current_user` is assigned/has access to.

If we wanted, we could even have a route like:
```
http://localhost:3000/users/9/associates/3
```
which would then drill into a connected associate's profile.  But there's a problem.....

If our consistent dashboard UX/UI is to hold for this route, then this route ought to act as a sort of miniature homepage for `associate_3` — presenting company/account/package info — filtered such that only material `user_9` has shared access to is visible… eg if `associate_3` had access to `company_32`, and `user_9` did not, then we would want to make sure that information for `company_9` remained protected at this route.

When trying to devise a strategy to accomplish this, I realized that much of this challenge wasn't just in articulating the filtering mechanisms in the controller, but also in how the views would need to adjust to the different frames of reference given the amount of data they were responsible for retrieving and rendering.  My original attempt to solve this began with an elaborate, but still rather standard, setup of view files and partials that accepted locals defined by instance variables.  As the controller's filtering mechanisms began to output different instance variables based on different `params`, I found it difficult to keep the partials and the ERB in the view files very DRY, and instead found abstracting out the variables in the controller and processing them entirely in a framework of helper methods as an easier path to pursue.

Although the app’s available routes changed by the time the app was completed, the approach of handling all the required filtering processes in the helper methods evolved to ultimately define the majority of the app’s HTML components.  As a result, view files are not used in this app.   Instead, two primary templates — one for displaying the logged-in content and one for signup/signin — are driven by a framework of helper methods that process a suite of abstracted instance variables from the controllers.

Instance variables are named to maintain consistency in the helper framework. For example, the `packages#new` and `accounts#new` produce the following instance variables:

	packages#new

	def new  
			@route_path = account_packages_path(route_params[:account_id])
			@nested_route_object = Account.find(route_params[:account_id])     
			@title = "New Package For #{@nested_route_object.kompany.name} >> #{@nested_route_object.name}"    
			@subject = @nested_route_object.packages.new
			@subject_class_attribute = Package.status
			@subject_collection_attribute = :status
			3.times {@subject.package_comments.build}      
		end



	accounts#new

	def new
			@route_path = user_accounts_path(current_user.id)
			@title = "Add New Account:"
			@subject = Account.new
			@subject.build_company
			3.times {@subject.packages.build}
			@child_class_attribute = Package.status
			@child_collection_attribute = :status
		end


These variables are used by the helper framework to ultimately populate the `form_structure` method.  This method builds out and renders the appropriate form based on the supplied variables.

 
	 def form_structure(subject)
 
	form_for(subject, url: @route_path) do |instance| 
      case strong_params[:controller]
      
		## New Associate form
		when "associates"

		## Invitation_form(subject)
			render partial: "dashboard_elements/invitation_template", locals: {:resource => subject}  

		## New Company form  
		when "companies"
			concat primary_subject(instance)
			concat tag.br      
			concat new_child(instance)
			concat tag.br      
			concat assign_new_associates(instance: instance, current_user: current_user)
			concat tag.br      
			concat submit_form(instance)


		## New Account form
		when "accounts"
			concat primary_subject(instance)
			concat subject.form_select_parent_label(instance: instance)   
			concat subject.form_select_parent(instance: instance, current_user: current_user, subject: subject)   
			concat tag.br      
			concat nested_new_parent(instance)
			concat tag.br      
			hidden_parent_tag(instance)
			concat new_child(instance)
			concat submit_form(instance)


		## New Package form
		when "packages"
			concat primary_subject(instance)
			concat primary_subject_collection_selection(instance)
			concat tag.br      
			hidden_parent_tag(instance)
			concat new_child(instance)
			concat submit_form(instance)

		## New Package Comment form
		when "package_comments"
			concat primary_subject(instance)
			hidden_parent_tag(instance)
			concat submit_form(instance)

			end
		end      
	end
 
The intended advantage of this approach is that if/when a new data model is added to the app, the helper framework will be able to render out the `newmodel#new` form with only the addition of the appropriate `when` clause to the `new_form` method’s case expression...  The developer will be able to simply select the appropriate combination of concatenated framework elements inside the clause, and the form will successfully render as long as the instance variables in its controller are labelled correctly.

The same approach is taken with presenting data in the app.  The kanban-style tiles in the app are designed to plug-n-play abstractedly labeled variables from multiple controllers.

	## Control contents inside of the columns tiles when called by the CANVAS method.
	
	def tiles(tiling_elements:, sorting_condition:)  
		case strong_params[:controller]
    
			when "companies"
				account = @company.accounts.find_by(name: sorting_condition)
				filtered_tile_collection = tiling_elements.specific(account)
				filtered_tile_collection.each {|element| concat tile_framework(element: element, variable: nil)}

			when "users" 
				filtered_tile_collection = tiling_elements.where(:status => sorting_condition).distinct
				filtered_tile_collection.each {|element| concat tile_framework(element: element, variable: nil)}

			when "associates"
				associates = @associates
				filtered_tile_collection = tiling_elements.merge(Company.where(name: sorting_condition)).distinct
				filtered_tile_collection.each {|element| concat tile_framework(element: element, variable: sorting_condition)}        
			end    
		end

The same intended functionality applies with the tiles as with the form framework....  Should new data need to be presented in the app, a simple addition to the case statement along with appropriately labelled controller variables ought to do the trick.  

Although this framework was not directly implemented in the user story conundrum outlined above (I ran out of time!) I believe this approach would help in solving the difficulty of presenting the `/users/9/associates/3` data without the need for bloated ERB code.  More thought would obviously need to be put into refining the framework to accept more versatile controller variables, especially in the spirit of having the framework serve as proper tool for the app that saves development time and keeps the codebase efficient.  

The process of working through building off of a conventional Rails framework, and then finding ways to manipulate the framework to solve unforeseen complications in the app, was very rewarding.  Combining both Rails helpers and conventional ruby code in the framework helped to consolidate my understanding of both Ruby and Ruby on Rails.  I recommend considering this design approach to any student wishing to push themselves in their Ruby/Rails learning journey.





 *A very real lesson I learned from attempting to build a data-rich dashboard app is how much the initial design blueprint matters.  Mapping out your nested routes—and fully understanding those routes implied data requirements—before starting to code the program is crucial.  You can get lost, as I did many times in the process of building this app, in myriad rabbit holes where you’re trying to shoehorn inappropriate data into inappropriate routes. 
*
