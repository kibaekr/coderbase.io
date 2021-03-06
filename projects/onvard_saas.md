---
title: Onvard Saas
permalink: onvard_saas
description: onvard_saas built on top of railskits saas
homepage: 
languages: [javascript]
visible: false
order: 
# Github Flavored Markdown reference
# https://help.github.com/articles/github-flavored-markdown
---


== Welcome to the SaaS Rails Kit

Hello Readme.

This Kit will help you get a quick start on building new web
applications that need a recurring billing component and have
multiple plan levels that are charged at varying rates. Typical
examples of this kind of Rails application are the services
offered by 37signals: Basecamp, Backpack, etc.

The Kit comes configured to allow customers to create paid
accounts with a 1 month free trial without collecting payment
information. Emails are sent to remind customers when a trial
account is about to expire, when the plan level gets changed, when
payment is made, and when an account is canceled.

Much of the functionality is in the saas-kit gem, not in this
sample app.  The installation instructions below include information
on how to install the gem.

== Getting Started

This code provides a complete, running Rails application. There
are a few things you need to do to get started, though. First,
edit the Gemfile, uncommenting the first line and changing the URL
to include your order ID. Then, run the following commands on the
command line:

newest ruby version: \curl -sSL https://get.rvm.io | bash -s stable --ruby

bundle install
./script/rails generate saas
rake db:create db:migrate saas:bootstrap


==

This will create your database (by default the app is configured
to use MySQL), load some sample data, and create a config file for
you in config/saas.yml (which you should review).

Once that's done, you are ready to go. Start up the app and browse
to it in your web browser. Log in with the information that was
output during the rake task, and you'll be logged in as the admin
user for a test customer account.

================================================================================
==* Additional Info from Onvard's Dev Team.
* Rmagick may need to be installed locally before running bundle install.
* Redis server needs to be installed and running: http://redis.io/topics/quickstart
* If not using pow, you will need to open up "/etc/hosts" file and add "127.0.0.1 www.localdev.com" & "127.0.0.1 beer.localdev.com" anywhere in that file. Then, you will be able to access the app in "beer.localdev.com:3000"
* Update(3.23.2014): You need to change the full_domain of each account record using rails console to "~~.localdev.com" 
* If you get "account needs to be confirmed first, go into rails console and do User.confirm! for each user 

==* For File Uploading
* run "redis-server" in terminal
* run "bundle exec sidekiq" in terminal

==* PRODUCTION

//login
ssh -i config/onvard.pem ubuntu@54.200.188.117
server restart: cap deploy:restart
cap deploy:rollback
logs: cap production tail_logs 
see: cap -T for all available commands



==* BACKUPS
https://console.aws.amazon.com/rds/home?region=us-west-2#dbinstances

==* TESTING
* rake db:test:clone 
* zeus start 
* in another terminal, run "guard"
* or just run "rspec" if zeus and guard is too difficult: http://railscasts.com/episodes/264-guard


==* STAGING

* cap staging deploy
ssh -i config/onvard.pem ubuntu@54.201.48.185

to wake up server: cap staging deploy:start


cap staging deploy:stop; cap staging deploy; cap staging deploy:migrate; cap staging deploy:start

================================================================================

== Deploying to a Production Server

This kit is designed to be served by a web server configured for
name-based virtual hosting. In my deployments, I point the
"default" configuration at the app so that traffic to any domain
name not explicitly set in my other virtual host sections gets
directed to the application.

For the "public" web site, where one typically has a welcome page,
tour, etc., you can add static page content to the views in
app/views/content. A catch-all route sends all requests to
/content/* to the ContentController, so
http://yoursite.com/content/about would be served from
app/views/content/about.html.erb.  

If you'd like to host your homepage separately (say you want to
host your main domain with Wordpress or something) you can create
a separate virtual host for www.yourdomain.com or yourdomain.com,
and forward all urls that begin with /signup to the Rails
application. You'll notice in config/routes.rb that the pricing
grid, account creation page, and sign-up thank-you page are all
served with urls that start with /signup. 

If the layout for your public site will differ from the layout of
the application, add a new layout to app/views/layouts and edit
app/controllers/accounts_controller.rb to change the new, plans,
and thanks actions to use that layout.

The application is written to allow all admins to update their
payment information while logged in to an account, which means
that SSL connections will be attempted for all the subdomains of
your base domain. Unless you want SSL certificate warnings for
your customers, then, you'll need to get a wildcard SSL
certificate. I use GoDaddy to purchase mine: currently $200 per
year. Of course you could rewrite the billing action in
app/controllers/accounts_controller.rb to redirect to a specific
subdomain to avoid the cost of a wildcard SSL cert... feel free to
do so. :)

<!-- TODO-->
Make sure you set up a cron job to run 'rake saas:run_billing' on a
daily basis. This script does the charging for account renewals
and sends notices of expiring trials.

== Integrating Into an Existing App

If you'd like to pull the billing functionality into your app that
you've already started, copy the relevant controller and model code
into your existing app, and include the necessary gems from the Gemfile
in your Gemfile. Then run 'bundle install', 'rails generate saas',
and 'rake db:migrate' to get started.  

== PayPal

PayPal payments are handled via PayPal's recurring payment profiles,
which are a part of the Website Payments Standard plan. You don't
need to sign up for Website Payments Pro or PayFlow -- just the
basic business plan. There are a couple of caveats when using PayPal
in addition to or in place of a standard credit card gateway:
 - You will need to enable API access in both your sandbox and
   live paypal accounts.
 - Put your sandbox credentials in the paypal stanza of the defaults
   section of config/saas.yml.  Put your live account credentials
   in the paypal section of the production stanza.
 - Your customers will be redirected to PayPal to collect billing
   information (of course), but they will also be redirected to
   PayPal when changing to a paid plan.  This is because a new
   recurring payment profile needs to be created with PayPal.  If a
   customer switches from a paid plan to a free plan, no redirect
   happens, as the existing recurring payment profile is simply
   destroyed behind the scenes.

== Stripe

This kit supports Stripe (stripe.com) via both their PCI compliant
gateway, and secure JavaScript API. In both cases, you need to
configure the gateway and credentials by setting "stripe" as the
gateway and your secret key as the "login".

Optionally, you may set the stripe_publishable_key value, which will
take advantage of the JavaScript API (leave blank to use just the
gateway). This method will use the same credit card form, but instead
of posting the credit card info to your servers (requiring you to
pass a higher-level PCI compliance standard), it will post the credit
card info directly to Stripe servers.  The Kit will then get a token
that it can use to bill the user's credit card.  In this manner, your
user's credit card info never touches your server.  When using the
javascript method, the credit card form will be disabled by default to
prevent submitting the user from submitting sensitive information to
your server when the user does not have a JavaScript capable browser.
The form is then enabled by JavaScript, and the payment information
will be submitted directly to Stripe.

== Contents

Hopefully you're familiar enough with the basics of Rails that I
don't have to explain what every file in this archive does. If
not, go read some books like Agile Web Development with Rails and
then come back here. With that out of the way, here are some of
the files you'll want to check out:

app/
  controllers/
    users_controller.rb - Here's an example of how you can use
    limits in your application. Notice the before filter to
    enforce the limit, and the call to inherit_resources to pull
    in generic RESTful methods. Also notice the
    begin_of_association_chain method, which is used to scope all
    the finds to the current account (the current_account method
    is defined in lib/subscription_system). Use this pattern
    throughout your application to make sure users only see the
    data associated with their account.

  models/
    account.rb - Near the top of the file you'll notice call to
    has_subscription, which is loads the saas gem and sets up the
    various limits you'll be checking for plan eligibility and for
    being able to do various things in your app. For example, the
    user_limit entry in the hash checks the count of associated
    users, and is used to create the reached_user_limit?
    convenience method in the plugin.
    
    user.rb - Basic User model with some of Devise's functionality
    overridden to make the login scoped by account (so you can
    have one user with the same email address belonging to
    multiple accounts).
    
  views/
    accounts/ - Views for updating billing, creating a new
    account, etc. are here
    
    content/ - Homepage content (splash page) and other content,
    like an about page, privacy policy, etc. go here.

config/
  saas.yml - Some settings for the application, fairly
  self-explanatory.  This is generated by the bootstrap task or
  the generator.
  
You'll find the bulk of the recurring billing functionality located
in the saas-kit gem.
  
== Testing

If you'd like to run the included test suite, run 'rake spec'.

For testing credit cards, you can use the VISA test number 4111
1111 1111 111 and the CVV 123.
