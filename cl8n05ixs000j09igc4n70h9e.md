# Setting up OAuth in Phoenix

So, it’s been like a month since I started learning Elixir and the Phoenix. Phoenix is a web framework powered by Elixir and the Erlang VM, which makes it fast.

In this post, we will see setting up “Sign in With Google” in a Phoenix application with [Ueberauth](https://github.com/ueberauth/ueberauth). We will be using the [ueberauth_google](https://github.com/ueberauth/ueberauth_google) hex package to perform this.

I’m going to assume that you have created the Phoenix app and created a user model. It should look similar to this.

```ex
# user.ex
schema "users" do
    field :email, :string
    field :first_name, :string
    field :last_name, :string
    field :provider, :string
    field :picture, :string

    timestamps()
  end
```

### Adding Dependencies

We will add the [ueberauth_google](https://github.com/ueberauth/ueberauth_google) dependency to the **deps** in the **mix.exs.** Now, the mix.ex file looks like this.

```ex
 defp deps do
    [
      {:phoenix, "~> 1.5.3"},
      {:phoenix_ecto, "~> 4.1"},
      {:ecto_sql, "~> 3.4"},
      {:postgrex, ">= 0.0.0"},
      {:phoenix_live_view, "~> 0.13.0"},
      {:floki, ">= 0.0.0", only: :test},
      {:phoenix_html, "~> 2.11"},
      {:phoenix_live_reload, "~> 1.2", only: :dev},
      {:phoenix_live_dashboard, "~> 0.2.0"},
      {:telemetry_metrics, "~> 0.4"},
      {:telemetry_poller, "~> 0.4"},
      {:gettext, "~> 0.11"},
      {:jason, "~> 1.0"},
      {:plug_cowboy, "~> 2.0"},
      {:ueberauth_google, "~> 0.9"}
    ]
end
```

Next we need to include this in the **application** function in the **mix.exs**. Now the application function looks like,

```ex
def application do
    [
      mod: {OAuthApp.Application, []},
      extra_applications: [:logger, :runtime_tools, :ueberauth_google],
    ]
end
```

After this run the following command to install the added dependencies. This will show the entire dependencies list including the ones we added now.

```ex
> mix deps.get
```

### Create a Google Project

Create a Google project by following the steps given [here](https://developers.google.com/identity/sign-in/web/sign-in). The Google app will generate the _client_id_ and _client_secret_ that we need to configure in our app. In the Google app configurations, set the redirect_uri as “[http://localhost:4000/auth/google/callback](http://localhost:4000/auth/google/callback%22)/”.

![](/svrmedia/heroes/image1-32.png)

### Updating Configurations

We will now add the configuration of our google app in our Phoenix application. Add the **client id**, **client secret,** and **redirect_uri** to the **config.exs** file.

```ex
# Ueberauth Config
config :ueberauth, Ueberauth,
  providers: [
    google: {Ueberauth.Strategy.Google, [default_scope: "email profile"]}
  ]
config :ueberauth, Ueberauth.Strategy.Google.OAuth,
  client_id: "741231234148352289-e9asdasdmomm9lar1brbundnmfg3j80f04ntqt.apps.googleusercontent.com", # Dummy code
  client_secret: "33uWw8WnPsadsdsSas7rjaTnkYbwu6", # Dummy code
  redirect_uri: "http://localhost:4000/auth/google/callback"
```

Now we have done all the configurations needed, we will get into coding now.

### Defining the Routes

We will add the routes for sign in with google. We need the define the following routes in the **routes.ex:**

* OAuth Initiate - This initiates the OAuth process.
* OAuth Callback - This route is the route defined as redirect_uri in the google app configuration. We will get the user details in this callback call.

We will add these routes with the scope “/auth”. The code to be added to the router.ex looks like where provide is Google here. We can also use the same routes for any other social sign we need.

```ex
scope "/auth", OAuthApp do
    pipe_through :browser

    get "/:provider", AuthController, :request
    get "/:provider/callback", AuthController, :callback
  end
```

### Auth Controller

We will now create a controller to handle the OAuth process.

* Inside the **controllers** folder, create a controller file called **auth_controller.ex.** We will define the callback function here.
* We need to define two callback functions.
  * OAuth Success Case
  * OAuth Failure Case

### OAuth Failure Case

The Ueberauth send the failure response in the below mentioned format. So, we need to define a callback function to match the failure pattern. If the “assigns” variable has the key “ueberauth_failure”, it indicates that the authorization failed.

```ex
def callback(%{assigns: %{ueberauth_failure: _fails}} = conn, _params) do
    conn
    |> put_flash(:error, "Failed to authenticate.")
    |> redirect(to: "/")
  end
```

### OAuth Success Case

If the failure key is not present then the authorization is successful and it sends the oauth response in the key “ueberauth_auth” in the response. So, we need to define the callback function to match the response pattern.

Like in the code below, we can match the user information and send it to the user changeset and based on the use case either create or update the user details.

```ex
def callback(%{assigns: %{ueberauth_auth: auth}} = conn, _params) do
    user_params = %{
      first_name: auth.info.first_name,
      last_name: auth.info.last_name,
      email: auth.info.email,
      picture: auth.info.image,
      provider: "google"
    }

    changeset = User.changeset(%User{}, user_params)

    case insert_or_update_user(changeset) do
      {:ok, user} ->
        conn
        |> put_flash(:info, "Thank you for signing in!")
        |> put_session(:user_id, user.id)
        |> redirect(to: Routes.lobby_path(conn, :index))

      {:error, _reason} ->
        conn
        |> put_flash(:error, "Error signing in")
        |> redirect(to: Routes.page_path(conn, :index))
    end
  end
```

Yay! Now, you have successfully used Google OAuth to authenticate your users. 

In this article, we discussed implementing Sign in with Google in our Phoenix application. Ueberauth provides OAuth for most of the social platforms such as Facebook, Twitter. WIth this we can also get the user token from the OAuth response and play with the Google APi to integrate google apps like Google Calendar, Google Meet. With this, the user does not have to remember a password which gives a good user experience.

If you find any difficulties, reach out to [heerthees@skcript.com](mailto:heerthees@skcript.com).