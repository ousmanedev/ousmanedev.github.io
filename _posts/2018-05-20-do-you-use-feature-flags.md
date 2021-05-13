---
title: "Do you use feature flags ?"
date: "2018-05-20"
coverImage: "ff.jpeg"
layout: post
---

### What are feature flags ?

When adding new features to production, problems often happen and their impact can range from simple bugs to a whole application shutdown.

Feature flags provide a way to: - Roll out features to specific users or groups of users instead of everyone - Enable/Disable features in real-time without altering your code or rolling back your release

### Usage of feature flags in a Rails application

#### Install and configure the `rollout` gem

The rollout gem is based on `Redis`, so we need to have it installed as well.

Add `gem 'redis'`  and `gem 'rollout'` to your `Gemfile`, and run `bundle install`.

Create an initializer file


# config/initializers/rollout.rb

$redis = Redis.new
$rollout = Rollout.new($redis)

#### Define our users group

Users groups are the list of users you'll be rolling out features to.

# config/initializers/rollout.rb
```ruby
$rollout = Rollout.new(Redis.new) 

$rollout.define_group(:admins) { |user| user.admin? }

$rollout.define_group(:testers) { |user| user.tester? }

$rollout.define_group(:pros) { |user| user.membership == 'pro' }
```

#### Roll out a new chat feature

Add the feature related code.

# First, check if the current user has access to the feature
if $rollout.active?(:chat, current_user)
  # chat feature code
end

Let's say we first want our testers only, to have this feature.  Run the following code in the rails console: `$rollout.activate_group(:chat, :testers)` Now, only the testers can access the new chat feature.

What if we now want to release the feature to our paying users ? Head to the rails console again and run this: `$rollout.activate_group(:chat, :pros)`. That's it. The pros users can now access the chat feature.

A problem ? Let's disable that feature until we fix it. Run the following code in the console, and you're done: `$rollout.deactivate_all(:chat)`

Learn more about the `rollout` gem [here](https://github.com/fetlife/rollout).

### Feature flags libraries in other languages/frameworks

- Python: [https://github.com/asenchi/proclaim](https://github.com/asenchi/proclaim)
- PHP: [https://github.com/opensoft/rollout](https://github.com/opensoft/rollout)
- Clojure: [https://github.com/yeller/shoutout](https://github.com/yeller/shoutout)

Thank you for reading.
