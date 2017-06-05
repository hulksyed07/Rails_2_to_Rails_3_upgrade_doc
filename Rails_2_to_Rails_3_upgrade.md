Upgrading from Rails 2 to Rails 3
----------------------------------------------

1. The best approach to avoid any confusion is build a fresh Rails 3 app and then keep on moving files
   from the old app to new app.

   You can use 'Meld' tool to compare the two directories and move the files one by one as per your wish:

     http://meldmerge.org/

2. Replace 'RAILS_ROOT' to Rails.root
3. Replace 'RAILS_ENV' to Rails.env
4. Replace <% form_for ... %> with <%= form_for %>. Without the equals sign the form will never get displayed on the page.
5. Replace 'RAILS_DEFAULT_LOGGER' with Rails.logger
6. A sample replacement of routes as per Rails 3 syntax is shown below:
  
  Rails 2
  -------------
    ActionController::Routing::Routes.draw do |map|

      map.admin_home '/admin', :controller => 'people', :action => 'index'

      map.namespace :admin do |admin|
        admin.resources :abc
        admin.resources :def,
          :collection => {
          :dummy_action => :get,
          :show_profile => :get
        }
      end

    end

  Rails 3
  -------------
    YourApplicationName::Application.routes.draw do

      get '/admin', :to => 'people#index', :as => 'admin_home'

      namespace :admin do
        resources :abc
        resources :def do
          collection do
            get 'dummy_action'
            get 'show_profile'
          end
        end
      end

    end


7. Replace 'activerecord' with 'active_record'. Remember this is case sensitive. Don't replace 'ActiveRecord'.

8. Most of the Rails 2 Apps dont have a gemfile and these functionalities remain in vendor/plugins.
   In Rails 3 you have a Gemfile.
   Try to identify the name of the plugins and search them on Internet. You will be able to find most of the gems.

9. Some of the gems are hosted on rubgems.org and the remaining you can directly use from github.com as shown below:

   Using from rubygems.org:
   ------------------------

      gem 'xyz', '1.0.0'

   Using from github.com
   ------------------------

      gem 'abc', '1.1.0', :git => 'https://github.com/rails/prototype_legacy_helper.git'

10. You may find difficulties in installing some of the gems from github.com

    The Primary reason may be because these gems do not have a '.gemspec' file.

    In such case you can fork that repository and add this file so as to make the gem usable in gemfile.

11. Some of the gems may be incompatible with Rails 3. Find an alternative for those gems on Internet.
    At last if you are not able to find any alternative for that gem, then fork that gem and try to apply some patches on it.
    Hope that works... In my case it worked :-)

12. if your application user Prototype Js and has methods like 'form_remote_for', 'link_to_remote', 'observe_field',
    etc in views then you can use the gem 'prototype_legacy_helper' as shown below:

        gem 'prototype_legacy_helper', '0.0.0', :git => 'https://github.com/rails/prototype_legacy_helper.git'

    By using the above gem the above methods will continue to work in Rails 3 too.

    You also need gem 'prototype-rails' in order to make prototype js work as shown below:

        gem 'prototype-rails', '3.2.1'

13. Add the attributes of a model that should be accessible for mass assignment as shown below:
  
      class Profile < ActiveRecord::Base

        attr_accessible :name, :email, :age
      
      end

    If you don't do this you will get 'cannot mass-assign protected attributes' error while saving the data from a form.

14. If your application uses 'page' as a parameter in find condition then you need to remove that.
    Rails 3 does not support 'page' parameter in find condition.
    You can use will_paginate gem and try to paginate it in controller and not in model as shown below:

    Rails 2
    ----------

      class Profile < ActiveRecord::Base

        def self.get_limited_profiles

          find(:all, :conditions => "inactive_flag <> 'Y'", 
                     :order => "created_at",
                     :page => {:size => 3, :current => 1})
        end

      end

    Rails 3 (If using will_paginate)
    ------------

      class Profile < ActiveRecord::Base

        self.per_page = 3
      
      end

      class ProfilesController < ApplicationController

        def index
            @profiles = Profile.paginate(:page => params[:page])
        end

      end

15. If your helpers return links and you are trying to display that in views then you may not be able to see those links.

    The reason for this is Rails 3 is more secure than Rails 2.

    Use '.html_safe' in views wherever you are trying to display a html_content which is not written directly in the view and comes from a helper or comes from the database as shown below:

    Helper
    --------

        module ApplicationHelper

          def get_archive_link user
            if user.is_archived?
              link_to 'Archive', user_archive_path(user)
            else
              link_to 'Retrieve', user_retrieve_path(user)
          end

        end

    View
    ----------

        The below line will not get displayed on the page:

          <%= get_archive_link(@user) %>

        Using html_safe will solve the issue

          <%= get_archive_link(@user).html_safe %>
    

16. Move the javascripts, stylesheets and images from 'public' folder to 'app/assets' folder.

17. Replace the full path for images in stylesheets with the name of image only as shown below
    
    Rails 2
    ----------

        background-image: url('../images/happy.gif');

    Rails 3
    -----------

        background-image: url('happy.gif');


18. If you have 'error_messages_for' helper used in your forms then you need to remove them or rewrite
    them as they are no longer available in Rails 3.
    In case you want to rewrite them you can use the below way:

    View
    ---------------

        <%= error_messages_for 'profile', "Following errors occurred:" %>

    Helper
    ---------------

        def error_messages_for(obj, message)

          object = instance_variable_get("@#{obj}")

          if object.present? && object.errors.any?

            error_messages = object.errors.full_messages.map {|msg| content_tag(:li, msg) }.join.html_safe
            contents = ''
            contents << content_tag(:p, message) unless message.blank?
            contents << content_tag(:ul, error_messages)
            content_tag(:div, contents.html_safe, :class => 'error')

          end

        end

19. Replace "set_table_name 'my_table'" to "self.table_name = 'my_table'" in Models.
20. Replace "set_primary_key 'profile_id'" to "self.primary_key = 'profile_id'" in Models.




