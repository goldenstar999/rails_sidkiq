Rails Sidkiq
================

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

This application was generated with the [rails_apps_composer](https://github.com/RailsApps/rails_apps_composer) gem
provided by the [RailsApps Project](http://railsapps.github.io/).

Rails Composer is supported by developers who purchase our RailsApps tutorials.

Problems? Issues?
-----------

Need help? Ask on Stack Overflow with the tag 'railsapps.'

Your application contains diagnostics in the README file. Please provide a copy of the README file when reporting any issues.

If the application doesn't work as expected, please [report an issue](https://github.com/RailsApps/rails_apps_composer/issues)
and include the diagnostics.

Ruby on Rails
-------------

This application requires:

- Ruby 2.3.1
- Rails 5.1.4

Learn more about [Installing Rails](http://railsapps.github.io/installing-rails.html).

Getting Started
---------------

Documentation and Support
-------------------------

### Gemfile
``` shell
gem 'sidekiq-scheduler'
```

### config.ru
``` ruby
require 'sidekiq/web'
require 'sidekiq-scheduler/web'

run Sidekiq::Web
```

### config/initializers/redis.rb
``` ruby
uri = URI.parse(ENV["REDISTOGO_URL"] || "redis://localhost:6379/" )
$redis = Redis.new(:host => uri.host, :port => uri.port, :password => uri.password)
```

### config/initializers/sidekiq_scheduler.rb
``` ruby
require 'sidekiq/scheduler'

redis_url = ENV['REDISTOGO_URL'] || "redis://localhost:6379/"

Sidekiq.configure_server do |config|
  config.redis = {
    :url => redis_url,
    :namespace => 'xx_server'
  }
end

Sidekiq.configure_client do |config|
  config.redis = {
    :url => redis_url,
    :namespace => 'xx_server'
  }
end

Sidekiq.configure_server do |config|
  config.on(:startup) do
    Sidekiq.schedule = YAML.load_file(File.expand_path('../../../config/scheduler.yml',__FILE__))
    Sidekiq::Scheduler.load_schedule! 
  end
end

```

### config/application.rb
``` ruby
...
config.active_job.queue_adapter = :sidekiq
```

### config/scheduler.yml
``` yaml
resource_worker:
  # cron: "1 * * * *"
  class: ResourceWorker
  every: '1m'
  queue: resource_worker
```

### config/routes.rb
``` ruby
require 'sidekiq/web'
require 'sidekiq-scheduler/web'

Rails.application.routes.draw do
  ...

  mount Sidekiq::Web, at: '/sidekiq'
end
```

### app/workers/resource_worker.rb
``` ruby
require 'sidekiq-scheduler'

class ResourceWorker
  include Sidekiq::Worker
  
  def perform
    users = User.all
    users.each do |user|
      user.wood += 3
      user.save
    end
  end
end
```

### Procfile
``` shell
web: bundle exec unicorn -p $PORT
worker: bundle exec sidekiq -c 5 -v -q resource_worker
```

Issues
-------------
You should add RedisToGo addon on heroku
https://stackoverflow.com/questions/13770713/rails-starting-sidekiq-on-heroku

You need to write Procfile to auto-run this sidekiq scheduler as background process on Heroku
Note that should add "-c -5 -v" options
``` shell
bundle exec sidekiq -c 5 -v -q resource_worker
```

Run
----------------
``` shell
bundle install
rails s
```
On other console window, run this command
``` shell
bundle exec sidekiq -q resource_worker
```
or 
``` shell
bundle exec sidekiq -c 5 -v -q resource_worker
```

Web page
------------
> http://localhost:3000

> http://localhost:3000/sidekiq

