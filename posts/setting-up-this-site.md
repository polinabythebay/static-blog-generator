title: Creating a blogging site with Flask and Github Pages
date: 2014-03-15
published: true 

There are many ways to go about hosting a personal blog/landing page. One of my favorite is via Github pages. It's free and there are options just as Jekyl or Octopress to create an out of the box site for you. There's also the option of rolling your own to deploy on Github pages, which I'll touch on.

http://jekyllrb.com/

The one main thing you need to take into consideration when making a site for Github pages is that it needs to be a static site. This means the site is composed entirely of HTML, CSS, and JavaScript and it needs to be able to be hosted without any server-side software other than a traditional web server. There's no database you can hook into or Python code on the server side. So how would you go about this? At first it seems like you could just manually create all of your HTML/CSS/JS files yourself and then just commit them, but that's a pretty tedious endeavor and pretty disorganized.

What we need ideally is some way of gracefully handling two issues that web frameworks are meant to solve: routing and templates. In addition to that, we'll need to find a better way of generating these files and pushing the static content to Github. The reason using a web framework in general is a good idea is because they will handle HTTP reqeusts and responses for us, which will be handy. Some web frameworks also have certain architectual patterns built in as well, such as the MVC pattern in Django and Rails. 

Routing and Templates

The two big issues that come when building a web application are: how do you map the requested URL from the browser to the code that handles it (routes), and how do we create HTML dynamically (templates)? Those are the two questions we'll need an answer to, and every web framework handles it slightly differenty. 

That's where Flask comes in. Flask is a python microframework for developing web applications, and in my opinion it's a great way to play around with python and the web without having the overhead that comes with learning a heavyweight framework such as Rails or Django. Flask works like other Python web frameworks work: it receives HTTP requests, dispatches code that generates HTML, and creates an HTTP response with that content. This is how all major server-side frameworks work (excluding JavaScript frameworks).

Flask handles routes by hooking up a function to a requested URL via a route() decorator. Here's an example of routing to the base URL:

	@app.route('/') #base URL
	def index():
		return render_template('index.html', posts= blog.posts)

Flask also lets you create dynamic URLs:

	@app.route('/blog/<path:path>.html')
	def post(path):
		post = blog.get_post_or_404(path)
		return render_template('post.html', post= post)

That solves our routing issue. Next up, Flask uses Jinja2 for templating. One great thing about Jinja2 is that it will cache your templates so that subsequent requests with the exact same arguments are returned from the cache instead of re-rendered. 

Flask lets us create a site without having to worry about routing or templating, but what about generating static content? We will need to a way to make a "static site generator". Many of those exist, you can take a look at them here:

https://www.staticgen.com/

But we want a way to generate our own set of files from Flask. Can we do it? Yes!

Frozen Flask, http://pythonhosted.org/Frozen-Flask/ is a Flask extension that "freezes" your flask application into a set of static files. The purpose of my application will be to show a list of blog posts and a little bit about me on the home page. Flask is very flexible when it comes to structuring files, so here's how I went ahead and initially organized them:

blogging-site

	/build 
		#this is my build directly where I'm storing my Freezer() instance of my flask app and pushing this to github pages
	/posts
		#this is where my posts will be stored, in markdown
	/static
		#where my CSS files live
	/templates
		#where my HTML templates live
	CNAME #place to put my IP address for my site
	generator.py 
	#the python file that will show my routes, Flask instance, and generate my static files 
	+ plus any additional python files for adding functionality

It's pretty simple. If you take a look at the file structure of a Rails app, it's much much more complicated. I found when first learning about web development it was a easier to start off with something like Flask to get a better understanding of the kind of functionality that needs to go into a web application before diving into the Rails file structure. 

Notes:

Running Flask locally: 

	polina: static_site *$ python generator.py
 	* Running on http://127.0.0.1:8000/

Generating your Frozen instance:

 	polina: static_site *$ python generator.py build

Running your freezer instance locally: 

	polina: build (master)*$ python -mSimpleHTTPServer 8001
	Serving HTTP on 0.0.0.0 port 8001 ...

Pushing your static site to Github pages: 

	polina: build (master)*$ git add -A
	polina: build (master)*$ git commit -m "adding blog post"
	[master 78e2e97] adding blog post
	 3 files changed, 151 insertions(+)
	 create mode 100644 blog/setting-up-this-site.html
	polina: build (master)$ git push origin master
	Counting objects: 10, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (6/6), done.
	Writing objects: 100% (6/6), 3.93 KiB | 0 bytes/s, done.
	Total 6 (delta 3), reused 0 (delta 0)
	To https://github.com/polinadotio/polinadotio.github.io.git
	   7abfa60..78e2e97  master -> master
	polina: build (master)$ 


