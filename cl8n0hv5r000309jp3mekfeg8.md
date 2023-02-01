# Custom Domain Login in Ruby on Rails

### Why Custom Domain?

Let’s take a scenario, in which an application [https://app.example.org](https://app.example.org) has to be accessed by multiple organizations (for instance, Org1 and Org2) with their own domain such as [https://app.org1.org](https://app.org1.org) and [https://app.org2.org](https://app.org2.org). This can be achieved by introducing a custom domain (the domain in which the user wants to access the application) in the application.

To create a system with custom domain features, you have to maintain the session in a multi-domain environment. Handling sessions in this multi-domain environment is not simple, because the cookies are mapped to the domain to where it is set.

### **Setting up Custom Domain Session**

Let’s take an example where the URL of the application is [https://app.example.org](https://app.example.org) and it has to be accessed with [https://site.example1.org](https://site.example1.org).

We will be using device authentication in this application. When we login with email and password there will be no confusion as the host is the same across all the actions. Also, the session was set by the domain accessed for the login. But, when we use omniauth logins such as Google or GitHub, there will be a difficulty as the callback URL can’t be set dynamically.

To overcome this difficulty, we follow the below-mentioned steps:

* Set the device’s default user authentication to ‘omniauth’ authentication
* In the callback action, redirect the custom domain (as Host) with a unique token (to identify the user) to the custom actions
* In the custom action, the user is identified and logged in to the custom domain
* Now, the session is set by the custom domain and the user can access the entire application
* Here, the application (_app.example.org_) acts as the base for many other mini-applications (_site.example1.org, etc.,_).

Let’s see the above steps in detail with google login as an example. Let’s say that the Google Auth Callback URL is _app.example.org/users/auth/google_oauth2/callback_.

### Omniauth Origin

In Omniauth, we have a parameter in the request which gives the origin URL of the authentication process. Let’s say we have the Sign In With Google Button in the_ /user/sign_in _action. If the user clicks it from _app.example.org/user/sign_in,_ then this parameter will be the URL. This will be available in the following key <b>request.env['omniauth.origin']</b>. With this, we can find the host/domain of the request.

### Custom Domain Login in Practice

When the user login with the custom domain URL [`https://site.example1.org/user/signin`](https://site.example1.org/user/signin), the below Omniauth Callback action has to be performed:

* Authenticate the user in the Application Level (across the application).
* Find the Host from the Omniauth Origin. In this case, the host will be **_site.example1.org_**

  URI.parse(request.env\['omniauth.origin'\]).host)
* Redirect to the custom action with the host as omniauth origin, along with the unique token to identify the authenticated user. Let’s call this custom action as _social_login_. To identify the user, we will encrypt the email as the unique token (as a parameter). So the redirect URL can be generated as `"https://#{HOST}/user/social_login?key=#{user.encrypt_email}"` and the redirect URL will be “**https://site.example1.org/user/social_login?key=encrypted_key**”.
* In the Social Login action, we can identify the user by decrypting the email which was received as the parameter in the above step. After identifying the user, we can sign in the user with the device **_sign_in_** function. Now, the user’s session will be tied to the custom domain.
```ruby
def social_login
  if params["key"].present?
    @user = User.find_by(
     email: User.decrypt(params["key"])
    )
    if @user.present?
      sign_in @user
      check_private_organizaiton
    else
      redirect_to root_path, notice: "Please try again!"
    end
  else
    redirect_to root_path, notice: "Please try again!"
  end
end
```
I hope this blog gives you an idea to maintain a session with a custom domain! Try it out and let me know what do you think at [heerthees@skcript.com](mailto:heerthees@skcript.com)