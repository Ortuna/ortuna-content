----
title: First Post
date: May 26, 2013
published: true
----

Well it's been long overdue, but I have a place to put random things I pick up
day to day.  There are alot of times I wished that I saved or written up a quick
guide for myself and others on a particular subject.

#How this blog works
So today I'll post how this blog updates the content.  Firstly, it is made up of
two repositories.  One is the repository that runs the site 
[here](https://github.com/Ortuna/ortuna.com). The other is the content 
[here](https://github.com/Ortuna/ortuna-content).  A webhook from github.com 
updates the 'content' repository on the server.

#Post hook server
```ruby
require 'sinatra'
require 'grit'

set :bind, '0.0.0.0'
set :port, 9000

get '/' do
  Dir.chdir '../content'
  g = Grit::Repo.new('../content')
  g.git.reset({:hard => true}, 'HEAD')
  g.git.pull({}, "origin", "master")
end
```

As you can see, this server does a simple git pull on content when visiting the 
server @ port 9000

#Git + Datamapper

I've created [dm-gitfs-adapter](https://github.com/Ortuna/dm-gitfs-adapter) for 
another project, but this was a good use case for it.  Given a list of markdown
files, I want to be able to query them just like any other persistance engine.

##Article Model
```ruby
class Article
  include DataMapper::Gitfs::Resource
  resource_type :markdown
  property :title, String
  property :date, Date
  property :published, Boolean

  def uri
    base_path.gsub(/\.md/, '')
  end  
end
```

Now I can query the git repo(which is auto updating) like so with `Article.all`
or just the published items `Article.all(published: true)`


#Conclusion
The goal is to write atleast one blog post.  Or a collection of things I've done
during the week and consolidate them.
