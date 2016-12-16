---
layout: post
title:  BigCommerce API authentication
date:   2017-11-13 16:00:00
description: how to do authentication for BigCommerce API to access it's resources.
---
#### Background:
Recently I started using BigCommerce. I found very few resources about BigCommerce API authentication so here I am writing step by step process for that.

#### Step 1: Create Account at BigCommerce
refer <a href="login.bigcommerce.com"> BigCommerce </a> site and create account using your email address. BigCommerce will refer you as technical parter.

#### Step 2: Create an App
You need to create application at <a href="https://devtools.bigcommerce.com/my/apps"> BigCommerce DevTool Site</a>. You should be careful while filling point no 4, Technical Details. Keep following in your mind;
<ul>
  <li>
    Auth Callback URL should be secure i.e. HTTPS
  </li>
  <li>
    You should able to access server log of that URL. BigCommerce will send you temporarry code to that URL. You can get it by server log.
  </li>
</ul>

#### Step 3: Install your App
Now, go to your store adimn page. This is the same page you have landed on in step 1 after signup. Go to Apps > My Apps > My Draft App from left side navigator. Find your application there which you have created in above step. Now, You will get 'Learn More' link by hovering mouse over your app. Click on it. You will get install button, click on it. Then, you will be asked to confirm your installation, confirm it.

#### Step 4: Get Access Token
You will get temporary code via your Auth callback URL which you have provided in step 2. You can get useful data from your server log. It  includes temporary code, scope and store hash as a context, may look like this;

{% highlight javascript %}

Started GET "/?code=d3mb7lnt511jb1zl3o833ojeuy8f9r8&context=stores%2Fnffse6w1bi&scope=store_v2_content+store_v2_customers+store_v2_customers_login+store_v2_default+store_v2_information_read_only+store_v2_marketing+store_v2_orders+store_v2_products+users_basic_information" for 182.72.224.42 at 2016-12-13 14:24:27 +0530
Cannot render console from 182.72.224.42! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
Processing by TreesController#all_trees as HTML
  Parameters: {"code"=>"d3mb7lnt511jb1vl3o933ojeuy8f9r9", "context"=>"stores/nlxse6w1bi", "scope"=>"store_v2_content store_v2_customers store_v2_customers_login store_v2_default store_v2_information_read_only store_v2_marketing store_v2_orders store_v2_products users_basic_information"}

{% endhighlight %}

I am using my sample rails application to receive call back from BigCommerce. Copy temporary data and paste it somewhere.

Now, use any tool that can send post request to BigCommerce. I am using <a href="">Postman</a> for that. Generate following URL using data from server log.

{% highlight javascript %}

https://login.bigcommerce.com?client_id=4cdm6fjsg9inh7dl4http2g6m94qyw1&client_secret=bc8ai7j2qd6h3rfoiwbbvpl3lqsjxlk&code=d3mb7lnt511jb1vl3o933ojeuy8f9r9&scope=store_v2_content+store_v2_customers+store_v2_customers_login+store_v2_default+store_v2_information_read_only+store_v2_marketing+store_v2_orders+store_v2_products+users_basic_information&grant_type=authorization_code&redirect_uri=https://9a010f27.ngrok.io&context=stores/nlxse6w1bi

{% endhighlight %}

This URL includes following parameters:

<ul>
  <li>
    client_id : The Client ID for your app, obtained during registration.
  </li>
  <li>
    client_secret : The Client Secret for your app, obtained during registration.
  </li>
  <li>
    code : Temporary access code received in the GET request discussed above.
  </li>
  <li>
    scope : List of OAuth scopes received in the GET request discussed above.
  </li>
  <li>
    grant_type : Always use the following: authorization_code.
  </li>
  <li>
    redirect_uri : Must be identical to your registered Auth Callback URI.
  </li>
  <li>
    context : The store hash received in the GET request, in the format: stores/{_store_hash_}
  </li>
</ul>

Now make POST request to this URL. You will receive permanent access token in response. It may look like,

{% highlight javascript %}

{
  "access_token": "luxdeqp1j9dry3kfse3ocn882d4o6yh",
  "scope": "store_v2_content store_v2_customers store_v2_customers_login store_v2_default store_v2_information_read_only store_v2_marketing store_v2_orders store_v2_products users_basic_information",
  "user": {
    "id": 648788,
    "username": "mayurt20@gmail.com",
    "email": "mayurt20@gmail.com"
  },
  "context": "stores/nlxse1w1bi"
}

{% endhighlight %}

Aditionaly you will need client id and client secret for some api authentications. To get that go to https://devtools.bigcommerce.com/my/apps and find your app. You will find view client id link. By clicking on that you will have pop up which will have client id and client secret.
<ul>
  <li>
    UI has to know the `key` name that is used to store sidekiq response in Redis
  </li>
</ul>

You can find code here <a href="https://github.com/charusat09/sync_sidekiq">on GitHub</a>
