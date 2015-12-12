# Authorization with Cancan

## Overview

Hey, can I delete this file?

It's a surprisingly loaded question. Did I create the file? Do I own it? What does it mean to own a file? Can I write to the file? If I can write to a thing, can I delete it?

These are questions of authorization.

*Authentication* is determining who someone is. When I take money out of an ATM, the bank authenticates me with using two authentication factors: my PIN, and the secret key stored on the chip in my ATM card. Before I get on a plane, the TSA (the airline security organization in the U.S.) authenticates me by asking for my ID and using the extremely advanced facial recognition hardware embedded in human brains to verify that the ID matches my face. Usernames and passwords and credentials are all authentication things.

*Authorization* is determining if someone can do a particular thing. The bouncer at the club authenticates me by looking at my ID, and then authorizes me to enter by looking at the list (though I am more typically asked to leave the premises).

We have been looking at different modes of authentication. Now, we'll shift our focus, and start dealing with authorization: how do you describe a permissions model, and how do you implement it in Rails?

(Also, in case you were wondering, standard UNIX permissions let you delete a file you don't own and can't write to. You must, however, be able to write to the directory that contains the file.)

## Cancan

[CanCan] is a gem for Rails that provides a simple but quite flexible way to authorize users to perform actions.

In your controllers, it looks like this:

    def show
      @article = Article.find(params[:id])
      authorize! :read, @article
    end

Here we're calling CanCan's `authorize!` method to determine if the user can `:read` this `@article`. If they can't, an exception is thrown.

In your views, it might look like this:

    <% if can? :update, @article %>
      <%= link_to "Edit", edit_article_path(@article) %>
    <% end %>

Or maybe this:

   <% if cannot? :update, @article %>
     Editing disabled.
   <% end %>

To define the permissions, you write an `Ability` class, which goes a bit like this:

    class Ability
      include CanCan::Ability
      
      def initialize(user)
        if user.admin?
          # only admins can change things
          can :update, Article
        end
        # but anyone can read them
        can :read, Article
      end
    end

Your `Ability` class is passed an instance of your `User` model. In the initializer, you call `can` or `cannot` to define what this particular user can do.

`can` and `cannot` both take as arguments the action as a symbol and an ActiveRecord class, an instance of which is the target of the action. You can be more specific if you want to allow or disallow actions based on information in the models. For example,

    can :write, Article, owner_id: user.id

This will let the user write any article whose `owner_id` is her user id. (That is, any Article she owns).

    can :read, Article, :category => { :visible => true }

This will let the user `:read` any `Project` whose `category`'s `visible` column is `true`.

CanCan doesn't make any assumptions about how you've stored permission information in your user model. It's up to you to add the appropriate fields to support your authorization scheme.

## A simple scheme

Here is a basic CanCan Ability class for a message board.

The rules are: anyone can post. Registered users can edit their post after posting (but not delete it). Moderators can do anything to any post.

    class Abilility
      include CanCan::Ability

      def initialize(user)
        can :read, Post
	can :create, Post

        unless user.nil? # guest
	  can :update, Post
	end
	if user.admin?
	  can :manage, Post
	end
      end
    end

`:manage` is a special CanCan action which means "any action". So users with an admin column set to `true` can do anything to any Post.

## Resources
  * [CanCan]

[CanCan]: https://github.com/ryanb/cancan