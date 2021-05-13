---
title: "Use ActiveRecord::Enum for your enumerations"
date: 2020-05-24T15:43:13+02:00
layout: post
---
I've recently started working on a rails codebase, where I noticed something quite interresting.

Let's say we have a model called `Post`. A post can either be `draft` or `published`. This data is saved in the `status` column.
In programming, we say that `status` is an [enumeration](https://en.wikipedia.org/wiki/Enumerated_type), because it has a finite number of possible values  (draft and published).

In this post, I'll try to show you what is wrong with the way post status is handled in that codebase, and how we can do better using [ActiveRecord::Enum](https://guides.rubyonrails.org/active_record_querying.html#enums)

## Current approach

This is how, `status` is handled in that codebase:

```ruby
# Post
#  status: text

class Post < ActiveRecord::Base
  validate :status, inclusion: %w(draft published)
end
```
The status data is saved as a string in the database, and a model validation is implemented to ensure it's always a valid value.

## What is wrong ?
That approach for sure works well, but it has some drawbacks:
- Let's say your team have decided that `published` is not a really meaningful name, and would like to use `live` instead. You'll then have to do a migration and update all of the posts which status is equal to `published`.
  ```ruby
  Post.where(status: 'published').update(status: 'live')
  ```
  That would of course, come with some headaches, if your database contains thousands or millions of posts.

- Validate the status data ? Rails handles that for you with ActiveRecord::Enum
- Querying or updating a post status, Rails provides some [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) for you to do that, with ActiveRecord::Enum
- Fetching all of the posts with a specific status, Rails still in charge with some syntactic sugar
- What if instead of storing strings in the status column, you could just store some integers ? That would cost less in database storage. For sure, it won't be a great deal until you have millions of records.


Instead of handling an enumeration from scratch, using `ActiveRecord::Enum` provides you flexibility and very cool features you'll need.

## Introducing ActiveRecord::Enum
### Implementation
Let's handle the post status, a better way.
- Save post status as an integer instead of a string
  ```bash
  rails g migration AddStatusToProducts status:integer
  ```

  ```ruby
    class AddStatusToProducts < ActiveRecord::Migration[6.0]
      def change
        add_column :products, :status, :integer
      end
    end
  ```
- Declare post status an enum
  ```ruby
    class Post
      enum :status, { draft: 0, published: 1 }
    end
  ```

That's it. We are now handling our post status with ActiveRecord::Enum. Let's see now, what we have access to.

### Goodies
#### Query status
```ruby
# Check if post is draft

# Without ActiveRecord::Enum
post.status == 'draft'

# With ActiveRecord::Enum
post.draft?
```
If you don't like cool features ;), you can still query the status the old way, despite that status is now an integer.

#### Update status
```ruby
# Set post status to published

# Without ActiveRecord::Enum
post.update(status: 'published')

# With ActiveRecord::Enum
post.published!
```
Here again, you can still update the status the old way, despite that status is now an integer.

#### Validation
With ActiveRecord::Enum, no need to add any inclusion validation as it's handled automatically.
```ruby
  post.update!(status: 'some_undefined_status')

  # it raises ArgumentError, "some_undefined_status is not a valid status"
```

#### Scopes
One thing I love in rails models is scopes.
ActiveRecord::Enum automatically provides scopes for each of your enum values.
```ruby
Post.draft
# Returns the same values as Post.where(status: :draft)

Post.published
# Returns the same values as Post.where(status: :published)
```

#### Renaming
Because enums values are mapped to integers in the database, you can rename them without needing to update your database.
Thus, we can safely rename `published` to `live`:
```ruby
class Post < ActiveRecord::Base
  enum status: { draft: 0, live: 1 }
end
```

## Enums in depth
### Use a Hash not an Array
In this post, we've declared our status enum as a Hash.
However, it's possible but not recommended, to declare it as an Array.

With a Hash, we set explicitly the mapping between enum values and integers that will be saved in the database.
With an Array, that mapping is handled by Rails and is based on the enum values order in the Array.
```ruby
# Don't do this at home
class Post < ActiveRecord::Base
  enum :status, %i(draft published)
end
# When using an array, the mapping of enum values to integer is automatic and order based.
# draft => 0, published => 1
```
The array declaration works well, but it's error-prone.
If you or one of your teammate, changes the enum values order, or introduce a new value in the bad place, shit will happen.
```ruby
# Here, we interchange the positions of draft and published
class Post < ActiveRecord::Base
  enum :status, %i(published draft)
end
# The mapping is messed up, due to the order changes.
# published => 0, draft => 1
```
This simple and innocent looking code changes, just break our application. From now, draft posts will be seen as published, while published ones will be seen as drafts.
With a Hash declaration, the mapping is clear, and this error is unlikely to occur.

### Inside Raw SQL
Using Arel, it's very easy to query statuses.
```ruby
posts = Post.where(.......).where(status: 'draft')
```
What if we have to write raw SQL and can't just use arel goodies.
```ruby
posts = Post.where('status = ?', 'draft')
# Won't work and will even raise an error
```
This query won't work because the status column is an integer, hence it doesn't recognize `'draft'` as a valid value. We have to supply the integer value of the status. Here's a workaround:
```ruby
posts = Post.where('status = ?', Post.statuses['draft'])
# Post.statuses = { draft: 0, published: 1 }
```
This again, is one of the goodies of `ActiveRecord::Enum`. Thanks to `Post.statuses`, you don't have to hard code, the status integer value inside your query.

### Multiple enums inside a model
When using multiple enums, scopes and helper methods generated by Rails, can be confusing.
Let's handle the "Made In" country of a product as an enum.
```ruby
class Post
  enum :status, { draft: 0, published: 1 }
  enum :made_in, { fr: 0, us: 1 }
end
```
We now have 4 scopes generated for our enums
- `Product.draft`
- `Product.published`
- `Product.fr`
- `Product.us`

You'll agree with me that, these scopes names are very confusing. One of the purpose of programming in ruby and rails, is code eloquency and beautifulness.
Let's bring some eloquency to these scopes, by leveraging [enums suffixes and prefixes](https://github.com/rails/rails/blob/b9ca94caea2ca6a6cc09abaffaad67b447134079/activerecord/lib/active_record/enum.rb#L153).

We'll add `status` as a suffix for the status enum, and `made_in` as a prefix for the `made_in` enum.

```ruby
class Post
  enum :status, { draft: 0, published: 1 },  _suffix: true
  enum :made_in, { fr: 0, us: 1 }, _prefix: true
end
```

We now have the following scopes:
- `Product.draft_status`
- `Product.published_status`
- `Product.made_in_fr`
- `Product.made_in_us`

Now, the scopes are more clear and self explanatory.

### Indexes are your friends
You'll probably be querying your model a lot, based on the enums it contains. Thus, don't forget to add an index, so the queries can be faster and optimized.
To understand database indexes in depth, [check this article](https://www.toptal.com/database/sql-indexes-explained-pt-1).

## Wrapping Up
- Use ActiveRecord::Enums for handling your enumerations
- ActiveRecord::Enums provides flexibility, validations, scopes, and synctatic sugar for querying and updating
- Always declare enums as Hash to avoid mapping erros due to order changes in Array
- Leverage suffixes and prefixes for better clarity
- Don't forget to add database indexes for your enum columns