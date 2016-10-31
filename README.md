# Sessions Controller

## Overview

We've covered how cookies can be used to store data in a user's browser.

One of the most common uses of cookies is for login. In this lesson, we'll cover how to use the Rails session to log users in.

## Objectives
  1. Describe where session data is stored.
  2. Create a `Sessions` controller in Rails.
  2. Log a user in with it.

## How login works on the web

Nearly every website in the world uses what I am going to call the "wristband" pattern. A lot of nightclubs use this pattern as well.

You arrive at the club. The bouncer checks your ID. They put a wristband on your wrist (or stamp your hand). They let you into the club.

If you leave and come back, the bouncer doesn't look at your ID, they just look for your wristband. If you buy a drink, the bartender doesn't need to see your ID, since your wristband proves you're old enough to buy alcohol.

You arrive at [gmail.com](http://mail.google.com). You submit your username and password. Google's servers check to see if your credentials are correct. If they are, Google's servers issue a cookie to your browser. If you visit another page on gmail, or anywhere on google.com for that matter, your browser will show the cookie to the server. The server verifies this cookie, and lets you load your inbox.

## How this looks in Rails

Let's look at what the simplest possible login system might look like in Rails.

The flow will look like this:

   * The user GETs `/login`
   * The user enters their username. There is no password.
   * The user submits the form, POSTing to `/login`.
   * In the create action of the `SessionsController` we set a cookie on the user's browser by writing their username into the session hash.
   * Thereafter, the user is logged in. `session[:username]` will hold their username.

Let's write a `SessionsController` to handle these routes. This controller has two actions, `new` and `create`, which we'll map in `routes.rb` to `get` and `post` on `/login`.

Typically, your `create` method would look up a user in the database, verify their login credentials, and then store the authenticated user's id in the session.

We're not going to do any of that right now. Our sessions controller is just going to trust that you are who you say you are.

```ruby
class SessionsController < ApplicationController
	def new
		# nothing to do here!
	end

	def create
		session[:username] = params[:username]
		redirect_to '/'
	end
end
```

There's no way for the server to log you out right now. To log yourself out, you'll have to delete the cookie from your browser.

We'll make a very small login form for `new.html.erb`,

```html
<form method='post'>
  <input name='username'>
  <input type='submit' value='login'>
</form>
```

Ordinarily, we would use `form_for @user`, but in this example, we don't have a user model at all!

When the user submits the form, they'll be logged in!

## Logging out

The log out flow is even simpler. We add a `SessionsController#destroy` method, which will clear the username out of the session.

```ruby
def destroy
  session.delete :username
end
```

The most common way to route this action is to `post '/logout'`. This means that our logout link will actually be a submit button that we style to look like a link.

It's tempting, but don't attach this to a `get` route. HTTP specifies that `get` routes don't change anything—logging out is definitely changing something. You don't actually want someone to be able to put a link to `http://www.yoursite.com/logout` in an email or message board post. It's not a security flaw, but it's pretty annoying to be logged out because of mailing list hijinks.

## Conclusion

At its base, login is very simple: the user provides you with credentials in a POST, you verify those credentials and set a token in the `session`. In this example, our token was literally the username the user typed. In a more complex app, it would most likely be their user id.

## Resources
  * [Rails Tutorial Chapter 8 — Log in, log out](https://www.railstutorial.org/book/basic_login)

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/sessions_controller_readme' title='Sessions Controller'>Sessions Controller</a> on Learn.co and start learning to code for free.</p>
