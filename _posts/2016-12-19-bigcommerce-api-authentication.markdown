---
layout: post
title:  BigCommerce API authentication
date:   2016-12-19 8:30:00
description: how to do authentication for BigCommerce API to access it's resources.
permalink: /how_to_do_bigcommerce_api_authentication/
---
#### Background:
<br/>
Recently I started using BigCommerce. I found very few resources about BigCommerce API authentication so here I am writing step by step process for that. I am using API V3 of BigCommerce.

#### Step 1: Create Account at BigCommerce
<br/>
refer <a href="https://login.bigcommerce.com"> BigCommerce </a> site and create account using your email address. BigCommerce will refer you as technical parter.
<br/>

#### Step 2: Create an App
<br/>
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
<br/>
Now, go to your store adimn page. This is the same page you have landed on in step 1 after signup. Go to Apps > My Apps > My Draft App from left side navigator. Find your application there which you have created in above step. Now, You will get 'Learn More' link by hovering mouse over your app. Click on it. You will get install button, click on it. Then, you will be asked to confirm your installation, confirm it.
<br/>

#### Step 4: Get Access Token
<br/>
You will get temporary code via your Auth callback URL which you have provided in step 2. You can get useful data from your server log. It  includes temporary code, scopes and store hash as a context, may look like this;

{% highlight javascript %}

Started GET "/?code=d3mb7lnt511jb1zl3o833ojeuy8f9r8&context=stores%2Fnffse6w1bi&scope=store_v2_content+store_v2_customers+store_v2_customers_login+store_v2_default+store_v2_information_read_only+store_v2_marketing+store_v2_orders+store_v2_products+users_basic_information" for 182.72.224.42 at 2016-12-13 14:24:27 +0530
Cannot render console from 182.72.224.42! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
Processing by TreesController#all_trees as HTML
  Parameters: {"code"=>"d3mb7lnt511jb1vl3o933ojeuy8f9r9", "context"=>"stores/nlxse6w1bi", "scope"=>"store_v2_content store_v2_customers store_v2_customers_login store_v2_default store_v2_information_read_only store_v2_marketing store_v2_orders store_v2_products users_basic_information"}

{% endhighlight %}

I am using my sample rails application and <a href="https://ngrok.com/docs#bind-tls"> ngrok </a> to receive call back from BigCommerce. Copy temporary data and paste it somewhere.

Now, use any tool that can send post request to BigCommerce. I am using <a href="https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en">Postman</a> for that. Generate following URL using data from server log.

{% highlight javascript %}

https://login.bigcommerce.com/oauth2/token?client_id=4cdm6fjsg9inh7dl4http2g6m94qyw1&client_secret=bc8ai7j25d6h3rfoiwbbvpl3lqwjxlk&code=d3mb7lnt511jb1vl3o933ojeuy8f9r9&scope=store_v2_content+store_v2_customers+store_v2_customers_login+store_v2_default+store_v2_information_read_only+store_v2_marketing+store_v2_orders+store_v2_products+users_basic_information&grant_type=authorization_code&redirect_uri=https://9a010f27.ngrok.io&context=stores/nlxse6w1dd

{% endhighlight %}

This URL includes following parameters:

<ul>
  <li>
    <b>client_id</b>     : The Client ID for your app, obtained during registration.
  </li>
  <li>
    <b>client_secret</b> : The Client Secret for your app, obtained during registration.
  </li>
  <li>
    <b>code</b>          : Temporary access code received in the GET request discussed above.
  </li>
  <li>
  <b>scope</b>         : List of OAuth scopes received in the GET request discussed above.
  </li>
  <li>
    <b>grant_type</b>    : Always use the following: authorization_code.
  </li>
  <li>
    <b>redirect_uri</b>  : Must be identical to your registered Auth Callback URI.
  </li>
  <li>
    <b>context</b>       : The store hash received in the GET request, in the format: stores/{_store_hash_}
  </li>
</ul>

Now make POST request to above URL. You will receive permanent access token in response. It may look like,

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

Aditionaly you will need client id and client secret for some api authentications. To get that go to <a href="https://devtools.bigcommerce.com/my/apps"> https://devtools.bigcommerce.com/my/apps </a> and find your app. You will find view client id link. By clicking on that you will have pop up which will have client id and client secret.

#### Step 5 : Access APIs
<br/>
Now you are ready to access BigCommerce APIs. We will access product API as demo. For that send `GET` request to `https://api.bigcommerce.com/stores/nlxse1w1bi/v3/catalog/products?include=images,variants` with following details as a header.

<ul>
  <li>
    <b>Accept</b> : application/json
  </li>
  <li>
    <b>Authorization</b> : Basic
  </li>
  <li>
    <b>Content-Type</b> : application/json
  </li>
  <li>
  <b>X-Auth-Client</b> : This will be your client id, obtained during registration.
  </li>
  <li>
    <b>X-Auth-Token</b> : Your permanent access token which you obtained in step 4.
  </li>
</ul>

Here I am passing ?include=images,variants as an additional parameters to get images and variants information with this API. For more info please refer <a href="https://github.com/bigcommerce/api/blob/master/docs/v3-catalog.md#expanding-product-sub-resources-on-get">BigCommerce V3 API Documentation </a>

As a response you will get whole product information from <a href="https://gist.github.com/charusat09/dcda5f7778867c9c3e4e1ab54886c056"> BigCommerce </a>

If you have queries then I will happy to help you out. Please reach me out at mayurt20@gmail.com

<center><a href="https://mayurkumar.info" target="_blank">mayurkumar</a></center>
