---
layout: post
title:  Divise + Omniauth2.0
date:   2017-12-19 8:30:00
description: use devise with omniauth2
permalink: /how_to_use_devise_with_omniauth/
---
#### Background:
<br/>
Now a days, <a href=""> OAuth2 </a> is quite popular. We are not going to in depth of how it's work. We will develop application which will use devise and omniauth2 so, our application will have feature of signup using omniauth or devise.
#### Step 1: Create Account at Provider i.e. google
<br/>
<a href="https://en.wikipedia.org/wiki/List_of_OAuth_providers"> See all providers</a>.<br/><br/>
I am taking google as an example. or a full list of these providers, please check OmniAuth's <a href="https://github.com/omniauth/omniauth/wiki/List-of-Strategies"> list of strategies. </a>. Go to <a href="https://console.developers.google.com/apis/credentials?project=fluent-archway-108614"> google developer console </a> and register our application. Google will authorise it next time when it will ask authentications using google. Click on create "OAuth consent screen" link and fill in require information. After saving go to "Credentials" link and click on "Create credentials" link. It will have several options. Click on "OAuth client id". Follow the instructions and you will have "Client ID". It will require while configuring devise.
<br/>

#### Step 2: Install necessary gems.
<br/>
Few gems will be needed to implement this feature. Lets add these to our Gemfile and run `bundle` command to install them.
<ul>
  <li>
    gem 'devise'
  </li>
  <li>
    gem 'devise-bootstrap-views'
  </li>
  <li>
    gem 'omniauth'
  </li>
  <li>
    gem 'omniauth-google-oauth2'
  </li>
</ul>

#### Step 3: Setup Devise
<br/>
I am assuming that we know how to setup devise for authentication in rails app. <a href="https://github.com/plataformatec/devise#getting-started"> Follow this </a> to see how to setup devise.
<br/>

#### Step 4: Integrate Omniauth2 with devise
<br/>
For this we need to update our users table. We should add the columns "provider" (string) and "uid" (string) to your User model.

{% highlight javascript %}
  rails g migration AddOmniauthToUsers provider:string uid:string
  rake db:migrate
{% endhighlight %}

NOTE: we are using omniauth with devise so no need to add provider middleware again in `config/initializers/omniauth.rb`

Now, declare the provider in your `config/initializers/devise.rb`.
{% highlight javascript %}
  config.omniauth :google_oauth2, "Client_ID", "Client_Secret", "callback_url"
{% endhighlight %}

Client_ID, Client_Secret and callback_url are generated in step1. callback_url may look like `http://localhost:3000/users/auth/google_oauth2/callback`

This will configure our strategy, we need to make our model (e.g. app/models/user.rb) omniauthable:
{% highlight javascript %}
  devise :omniauthable, :omniauth_providers => [:google_oauth2]
{% endhighlight %}

we can add multiple providers. Devise is already setup so it will create the following url methods:

user_omniauth_authorize_path(provider)
user_omniauth_callback_path(provider)

we can use this path methods to generate signin link i.e. `<%= link_to "Sign in with Google+", user_google_oauth2_omniauth_authorize_path %>`.

By clicking on the above link, the user will be redirected to Google. (If this link doesn't exist, try restarting the server.) After inserting their credentials, they will be redirected back to your application's callback method. To implement a callback, the first step is to go back to our `config/routes.rb` file and tell Devise in which controller we will implement Omniauth callbacks:
{% highlight javascript %}
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
{% endhighlight %}

Now add the file `app/controllers/users/omniauth_callbacks_controller.rb` and implemente callback as an action with the same name as the provider. Here is an example action for our google provider that we could add to our controller:
{% highlight javascript %}
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def google_oauth2
    # implement the method below in your model (e.g. app/models/user.rb)
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
{% endhighlight %}

Implement from_omniauth method to our user model.
{% highlight javascript %}
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
    user.email = auth.info.email
    user.password = Devise.friendly_token[0,20]
    user.name = auth.info.name   # assuming the user model has a name
    user.image = auth.info.image # assuming the user model has an image
    # If you are using confirmable and the provider(s) you use validate emails,
    # uncomment the line below to skip the confirmation emails.
    # user.skip_confirmation!
  end
end
{% endhighlight %}

This method tries to find an existing user by the provider and uid fields. If no user is found, a new one is created with a random password and some extra information.

Finally, if you want to allow your users to cancel sign up with Facebook, you can redirect them to cancel_user_registration_path. This will remove all session data starting with devise. and the new_with_session hook above will no longer be called.


#### Step 5 : Logout links
<br/>
{% highlight javascript %}
devise_scope :user do
  delete 'sign_out', :to => 'devise/sessions#destroy', :as => :destroy_user_session
end
{% endhighlight %}
If you have queries then I will happy to help you out. Please reach me out at mayurt20@gmail.com

<center><a href="http://mayurkumar.info" target="_blank">mayurkumar</a></center>
