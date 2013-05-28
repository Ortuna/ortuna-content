----
title: Ember.js With Padrino
date: May 27, 2013
published: true
----
Recently I've been working on a project that is using Ember.js
as the heart of the application.  It was a mix of a mini-API and
a few controllers to serve up the front-end application.  This is
why I opted for [Padrino](http://padrinorb.com).  Let me highlight
a few steps to get the emberjs application a bit easier to work
with while not investing a whole lot of time on architecture (for
the first release).

#Application Structure
The first thing I didn't want was to have the application code be
in one single javascript file.  This would make it a nightmare to
maintain!  So I used the basic structure of:

```
  public/javascripts/lib
  public/javascripts/app
   - controllers
   - views
   - models
```

`controllers` and `models` would contain the individual controller and
model files e.g. `post.js` or `post_controller.js`. `views` contained
all the `.handlebars` templates. `lib` contained the actual ember,
handlebars and jquery libraries.

In the controller I had some bit of code to find all these files via
`Dir.glob`.

```ruby
...
  private
  def js_assets
    public_assets '**/*.js'
  end

  def handlebar_assets
    public_assets '**/*.handlebars'
  end

  def public_assets(pattern)
    path ||= Padrino.root('public', 'javascripts', 'app', pattern)
    [].tap do |paths|
      Dir.glob(path).each do |asset| 
        paths << asset.gsub(Padrino.root('public'), '')
      end
    end
  end
```

#Asset loading


##Padrino controller
Controller comprised of something like this:

```ruby
  get '/' do
    @js_assets  = js_assets
    @hbs_assets = handlebar_assets
    render :xyz
  end
  
```

##haml template
I had a partial for these javascript libraries:

```haml
= javascript_include_tag 'lib/handlebars-1.0.0-rc.3.js'
= javascript_include_tag 'lib/ember-latest.min.js'

- if @hbs_assets
  :javascript
    var templates = #{@hbs_assets.to_json};
    templates.forEach(function(template){
      $.ajax({
        url: template,
        async: false
      }).success(function(data) {
        var templateName = basename(template, '.handlebars');
        Ember.TEMPLATES[templateName] = Ember.Handlebars.compile(data);
      });
    });

    function basename(path, suffix) {
      var b = path.replace(/^.*[\/\\]/g, '');

      if (typeof(suffix) == 'string' 
       && b.substr(b.length - suffix.length) == suffix) {
        b = b.substr(0, b.length - suffix.length);
      }

      return b;
    }
    
- if @js_assets
  - @js_assets.each do |asset|
    = javascript_include_tag asset
```

##Ember.js javascript loading
The js_asset loading part of the template is straight forward.

```
  - if @js_assets
    - @js_assets.each do |asset|
      = javascript_include_tag asset
```
outputs the `javascript_include_tag asset` for every asset.  This
will load anything that is inside `public/javascripts/app/**/*.js`
(application.js, application_controller.js, etc..)
without us having to take any extra steps.

##Handlebars template loading
The notable part of the loading is the handlebars templates.
All .handlebars file paths are pushed into the `@hsb_assets` array.
This array is then converted into JSON via `var templates = #{@hbs_assets.to_json};`
and inserted into a javascript for the client side.  When the client
executes this prepared script, an AJAX request is made to each of the 
.handlebars templates and downloaded.

###Handlebars and Emberjs
I should mention there are multiple ways of loading handlebars templates into 
Ember.js.  One being you embed your templates in the HTML of your Padrino view

```
<script type="text/x-handlebars" id="index">
  <h1>People</h1>

  <ul>
  {{#each model}}
    <li>Hello, <b>{{fullName}}</b>!</li>
  {{/each}}
  </ul>
</script>
```

The name of this template is `index`.

Another way of doing this is what the script above does:

```
Ember.TEMPLATES[templateName] = Ember.Handlebars.compile(data);
```

Ember.js can now pickup and display your template from Ember.TEMPLATES, which
I suspect is where it keeps your HTML rendered templates.

##Conclusion
This is a fast and easy way to get up and running with Ember.js on *development*.
For *Production* you would probably want to setup an asset pipline system( such as
[sinatra-assetpack](https://github.com/rstacruz/sinatra-assetpack)).  The net
result of all this is frictionless development with Ember.js.  You can add controllers
and views or any javascript that you need and it will just work!
