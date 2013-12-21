---
layout : post
title: "Tailing log files with Capistrano 3"
date: 2013-12-21 21:37:00
categories: rails devops
biofooter: false
bookfooter: true
---
When deploying Rails Applications with Capistrano 2 it was common to
have tasks to tail log files on production servers so that
they could be viewed locally without sshing into the remote
machine(s). In this post I'll cover how to do this with Capistrano 3.

In Capistrano 2, our streaming code would look something like this:

``` ruby
namespace :logging do
  desc "tail rails logs"
  task :tail_rails, :roles => :app do
    trap("INT") { puts 'Interupted'; exit 0; }
    run "tail -f #{shared_path}/log/#{rails_env}.log" do |channel, stream, data|
      puts "#{channel[:host]}: #{data}" 
      break if stream == :err    
    end
  end
end
```

In Capistrano 3, it's even simpler:

``` ruby
namespace :logging do
  desc "tail rails logs" 
  task :tail_rails do
    on roles(:app) do
      execute "tail -f #{shared_path}/log/#{fetch(:rails_env)}.log"
    end
  end
end
```

The above task should be added to `lib/capistrano/tasks/logs.cap` and
can be invoked with:

``` bash
cap production logging:tail_rails
```

And stopped with `ctrl c`.

This will tail the log files from all hosts defined as having the role
`:app`.
