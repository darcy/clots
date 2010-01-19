# Clots

This project seeks to implement extensions for liquid whereby it has the power of other template libraries.

One of the big benefits of liquid is that it enforces a strict MVC paradign where the V cannot affect the M.  This is done for security reasons, but is an excellent approach to coding views in general.  Therefore, we seek to make liquid a fuller template library so it can be used for all views, not just ones that joe user can modify.

## Form Builder

Clots allows a form to be created like so

{% formfor recipe %}
{{ errors }}
  <p>
  	Title: {% field :title %}
  </p>
  <p>
    Description: {% text :description, rows:4, cols:40 %}
  </p>
  <p>
    {{ "Update" | submit_button }}
  </p>
{% endformfor %} 

And generate:

<form method="POST" action="/recipes/1"><input type="hidden"
name="_method" value="PUT"/><input name="authenticity_token"
type="hidden" value="31b0e7e9d18e01f0733225060dbcfd06423f1832"/>
  <p>Title:<input type="text" id="recipe[title]" name="recipe[title]" value="wsws"/>
  </p>
  <p>Description:<textarea name="recipe[description]" rows="4"
cols="40">description of item</textarea>
  </p>
  <p>
    <div class="form-submit-button"><input type="submit"
value="Update"/></div>
  </p>
</form> 

If there were errors, they would both appear at the top of the form and wrap the invalid form items.  You'll note also that CSRF protection is added if enabled.

## BaseDrop Class

In order for everything to work correctly, it is necessary that your drops inherit from our Clots::BaseDrop class.  BaseDrop is pretty much ripped out of the Mephisto project.

Your Drops inheriting from it can then add additional attributes, just like in Mephisto:

class BookDrop < Clot::Base
  liquid_attributes << :title << :author_id << :genre_id
end

would provide a drop with access to the title, author_id and genre properties of the underlying ActiveRecord.

We also added a few extra methods to the BaseDrop class (as well as taking some out that were specific to Mephisto):

    def id
      @source.id
    end

    def dropped_class
      @source.class
    end

    def errors
      @source.errors
    end 

This is necessary for having the BaseDrop and its subclasses interact properly with our form builder and filters.  You would probably be fine just adding these methods to your current drops as well.

## to_liquid added to ActiveRecord::Base

We made the to_liquid method a little DRYer, favoring convention over configuration.  to_liquid is now automatically added to the ActiveRecord::Base class, and - unless overridden - works as follows:

a) When to_liquid is called on a model, it searches for a class of the same name with "Drop" appended to it. (obviously you'd have to have a drop folder somewhere in your path)
b) In cases of Single-Table-Inheritance, it follows the inheritance chain until it finds the appropriate drop.  So if you have an Admin model that inherits from a User model, will use UserDrop if no AdminDrop exists.
c) It then instantiates the appropriate drop class, with the active record as a parameter to the drop's constructor.

We thought this would be better than explicitly throwing to_liquid into the model through my "acts_as_liquid" or explicitly adding "to_liquid" to each model.  Philosophically speaking, we don't think models should contain any code that exists only to deal with the views.

## content_for and yield tags

Tags have been defined to provide similar functionality to rail's 'content_for' and 'yield' statements.  

The 'yield' tag is similar in function to liquid's 'include' tag, however the template name is automagically prefixed with the current controller and view directories.  This means that rather than defining a content_for tag in a view, the tag should be placed in a sub-folder of the view named after the action it will be called from.  Using the yield tag without any arguments will insert the content_for_layout variable, so it can be used the same as a typical yield statement.

The if_content_for block simply checks to see if a given template file exists and then outputs its contentents if so.

In order to use either of these tags (or the include tag) something similar to this will need to be added as a before_filter on your controller

  Liquid::Template.file_system = Liquid::LocalFileSystem.new( MyController.view_paths )
  

## Filters for RESTful routes

We added some filters for restful routes.  They are contained within the url_filters directory.

## Test Cases

We have tried to write tests for all aspects of our plugin.  Reading the tests is a good way to learn about how everything works.

Copyright (c) 2008 Ludicast