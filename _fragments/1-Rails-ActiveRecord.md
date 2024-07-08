---
layout: fragment
title: Debugging Rails ActiveRecord PostgreSQL Injection
tags: [Rails, ActiveRecord, SQL Injection]
description: Setting up an environment to debug Rails ActiveRecord PostgreSQL Injection attacks
keywords: ActiveRecord, SQLi
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# Debugging Rails ActiveRecord PostgreSQL Injection
During engagements, we might encounter a web application that is potentially vulnerable to SQL injection and require further investigation. If we have the source code, we can attempt to replicate this locally.

## Setting Up Kali Environment
Update Kali environment
```
sudo apt update
```

Install PostgreSQL
```
sudo apt install postgresql postgresql-contrib
```

Start and Enable PostgreSQL
```
sudo systemctl start postgresql
sudo systemctl enable postgresql (optional)
```

Change to Postgres user
```
sudo -i -u postgres
psql
  ALTER DATABASE template1 REFRESH COLLATION VERSION; (fix collation issues)
  CREATE DATABASE testdb; (create database)
  \c testdb
  CREATE TABLE test ();
  INSERT INTO test () VALUES ();
  \q
```

Set Up Ruby and Required Gems
```
sudo apt install ruby ruby-dev
gem install bundler

[Create a new directory]
bundler init
nano Gemfile
  "gem "pg"
  gem "activerecord"

bundle config set --local path 'vendor/bundle' (Install locally)
bundle install
```

Script Execution
```
bundle exec ruby test_injection.rb
sudo systemctl restart postgresql (restart if required)
```

test_injection.rb
```
require 'active_record'
require 'pg'

# Database configuration
ActiveRecord::Base.establish_connection(
  adapter: 'postgresql',
  host: 'localhost',
  username: 'superuser',
  password: 'superuser',
  database: 'testdb'
)

# Define the model
class test < ActiveRecord::Base
end

# Define the test method
def test_function(params)


# Test cases
def run_tests
  puts "Test case: SQL Injection Attempt"
  begin
    sql_injection_test = test()
    puts "Job: #{sql_injection_test[:job]}"
    puts "Query: #{sql_injection_test[:query]}"
    puts "Result: #{sql_injection_test[:result].inspect}"
  rescue => e
    puts "Caught exception: #{e.message}"
  end
end

# Run the tests
run_tests
```
