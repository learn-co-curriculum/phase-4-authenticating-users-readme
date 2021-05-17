# Sessions Controller

## Learning Goals

- Understand how login works on most websites
- Follow REST conventions for handling session data

## Introduction

We've covered how cookies can be used to store data in a user's browser.

One of the most common uses of cookies is for login. In this lesson, we'll cover
how to use the Rails session to log users in.

## How Login Works

Nearly every website in the world uses what we like to call the "wristband"
pattern. A lot of nightclubs use this pattern as well.

You arrive at the club. The bouncer checks your ID. They put a wristband on your
wrist (or stamp your hand). They let you into the club.

If you leave and come back, the bouncer doesn't look at your ID, they just look
for your wristband. If you buy a drink, the bartender doesn't need to see your
ID, since your wristband proves you're old enough to buy alcohol.

You arrive at [gmail.com](http://mail.google.com). You submit your username and
password. Google's servers check to see if your credentials are correct. If they
are, Google's servers issue a cookie to your browser. If you visit another page
on gmail, or anywhere on google.com for that matter, your browser will show the
cookie to the server. The server verifies this cookie, and lets you load your
inbox.

## How This Looks in Rails

Let's look at what the simplest possible login system might look like in a Rails
API/React application.

The flow will look like this:

- The user navigates to a login form on the React frontend
- The user enters their username. There is no password (for now).
- The user submits the form, POSTing to `/login` on the Rails backend.
- In the create action of the `SessionsController` we set a cookie on the user's browser by writing their user ID into the session hash.
- Thereafter, the user is logged in. `session[:user_id]` will hold their user ID.

Let's write a `SessionsController` to handle these routes. This controller has
one action, `create`, which we'll map in `routes.rb` for `POST` requests to
`/login`:

```rb
post "/login", to: "sessions#create"
```

Typically, your `create` method would look up a user in the database, verify
their login credentials, and then store the authenticated user's id in the
session:

```rb
class SessionsController < ApplicationController
  def create
    user = User.find_by(username: params[:username])
    session[:user_id] = user.id
    render json: user
  end
end
```

There's no way for the server to log you out right now. To log yourself out,
you'll have to delete the cookie from your browser.

Here's what the login component might look like on the frontend:

```js
function Login({ onLogin }) {
  const [username, setUsername] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    fetch("/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ username }),
    })
      .then((r) => r.json())
      .then((user) => onLogin(user));
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

When the user submits the form, they'll be logged in! Our `onLogin` callback function
would handle saving the logged in user's details in state.

## Logging out

The log out flow is even simpler. We can add a new route for logging out:

```rb
delete "/logout", to: "sessions#destroy"
```

Then add a `SessionsController#destroy` method, which will clear the username
out of the session:

```rb
def destroy
  session.delete :user_id
  head :no_content
end
```

Here's how that might look in the frontend:

```js
function Navbar({ onLogout }) {
  function handleLogout() {
    fetch("/logout", {
      method: "DELETE",
    }).then(() => onLogout());
  }

  return (
    <header>
      <button onClick={handleLogout}>Logout</button>
    </header>
  );
}
```

## Conclusion

At its base, login is very simple: the user provides you with credentials in a
by filling out a form, you verify those credentials and set a token in the
`session`. In this example, our token was their user id. We can also log users
out by removing their user ID from the session.
