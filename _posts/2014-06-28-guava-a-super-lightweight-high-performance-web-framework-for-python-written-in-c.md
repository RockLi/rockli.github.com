---
layout: post
title: "Guava - A super lightweight high performance web framework for Python written in C"
description: ""
category:
tags: []
---
{% include JB/setup %}

# Background

These days I'm on my vocation, so I decided to do something really special.

Finally I decided to write and release one open source project in the Python field.

Afterwards I got the idea to write a new web framework for Python but **not in the traditional** ways. There're
lots of web frameworks for Python in the world.

Maybe you guys will ask "Why you want to reinvent the wheel again?". The answer is very simple, "I'm not reinventing", I'm trying to bring some new ideas for writing new web frameworks.

I used to write a web framework in PHP about seven years ago while I'm still a student, and I understand the underlying implementation of some key components like session, router... in PHP for example.

So I started the [Guava project](https://github.com/flatpeach/guava). All of my following projects will build on top of Guava. Some of them will be opend sourced soon, not just serving as the example but also want to get the best performance for those applications.

# Main Features

## **Super lightweight while compared to Django**

From my perspective, one web framework should only do what it should do.

For example, users can choose SQLAlchemy as the ORM, Jinjia2 as the template engine... all of these components should be chosen by users nor forced by the framework designer.

So inside Guava, I only supply these components:

1. Builtin HTTP Sever

   The HTTP Server is built on top of libuv which NodeJS either. I don't want to still build on top of WSGI. The benifits of having the builtin HTTP server is to simplify the deployment and the development.

2. Router

   This component is really really important in all of the web frameworks. Unlike other web frameworks, I don't want users to manually register the router. I think convention is better than configuration, so inside Guava, all default routers has the automatic routing ability.

   You can even extend the Router to build advanced routers or register manually if you want.

   The guava project comes with three special builtin Routers and one Router for you particular requirements:

   * StaticRouter

     Serving the static directory which holds css,js,image files.


     For example, StaticRouter is mounted for /static, and my local static directory is my_static_directory


     	|     URL       |     Local File
     	| /static/1.css | my_static_directory/1.css
     	| /static/1.jpg | my_static_directory/1.jpg


     StaticRouter has one special parameter ```allow_index```, the default value is False.

     If this value is set to True, you can visit: xxx/static to list all files and sub directories in your current directory, just like ```python -m SimpleHTTPServer``` does, not including the performace part. I used some internal apis of libuv to optimize the performance.


   * RESTRouter

     Especially useful when you wanna supply RESTFul api.

     	| METHOD |  URL      |      Class      | Method
     	| GET    | /users    | UsersController | GET
     	| GET    | /users/10 | UsersController | GET_ONE
     	| DELETE | /users/10 | UsersController | DELETE
     	| POST   | /users/   | UsersController | POST
     	| PUT    | /users/10 | UsersController | PUT

   * MVCRouter

     This is the tradional way for one web framework to route the request.

     	|  URL           |       Class     | Method
     	|  /             | IndexController | index
     	|  /user         | UserController  | index
     	|  /user/message | UserController  | message


   * Router

     Give a list of router rules you want to register


     	router = guava.router.CustomRouter({
			'/about': guava.handler.Handler(package='.',
											module='about',
											cls='AboutController',
											action='index'),
			'/test': guava.handler.Handler(package='.',
										   module='about',
										   cls='AboutController',
										   action='test'),
			'/favicon.ico': guava.handler.RedirectHandler('/static/favicon.ico')
     	})


   All routers have one speciall parameter **mount_point**, which means this router only take effect under that URL.

   A simple example here:

		1. wwww.code-trick.com/
			This is your main website, you can use MVCRouter here.

		2. www.code-trick.com/api
			You supply the RESTFul service here, use the RESTRouter here

		3. www.code-trick.com/static
			All your static files, use the StaticRouter here

        4. www.code-trick.com/blog
			Your blog is here and different with the main website, maybe you can use the MVCRouter here


	All routers will construct a tree-like structure, you can mix several sub applications into one big application. Sample codes:


		server = guava.server.Server()

		router = guava.router.MVCRouter(mount_point="/")

		api_router = guava.router.RESTRouter(mount_point="/api")

		static_router = guava.router.StaticRouter(mount_point="/static", directory="./static")

		blog_router = guava.router.MVCRouter(mount_point="/blog")

		server.add_router(router, api_router, static_router, blog_router)


	Simple enough, right?

	By the way, you can customize whether to enable the session mechanism for one router. If you want, you can use different session for different mount point.

	If you want to build complex routing rules or register the router manually, you can do like this:

		router = guava.router.Router({
			'/about': guava.handler.Handler(...)
		})

	or build your advanced router:


		class MyAdvancedRouter(guava.router.Router):
			def __init__(self, *args, **kwargs):
				super(MyAdvancedRouter, self).__init__(*args, **kwargs)
				self.register(...)

			def route(self, req):
				if req.method == 'GET' and req.path == '/about':
					return guava.handler.Handler(....)

				return None

	If all of above routers still can not match your requirements, please send your feature request directly on the Github issues or to my email: __insfocus BIGAT gmail.com__. I'm glad to implement it in Guava if it's appropriate.


3. Session

	The session is especially useful if you want to build auth based applications.

	Normally like PHP, the default session is File based. In guava, I introduced InMemory session store and File based store, and introduced one way for you to extend. Maybe you want to store session data to other brokers like Redis, Memcache, PostgreSQL, MySQL, MongoDB...

	The reason why I didn't integrate Redis based session store in Guava is I don't want the redis client library be one of the dependicies of Guava.

	* Initialize the memory based session store

			session_store = guava.session.SessionStore(type=guava.session.Mem)

	  This is especially useful during your development.

	* Initialize the file based session store

			session_store = guava.session.SessionStore(type=guava.session.File)

	  For small to mediam sized web applications, you can choose the file based solution.

	* Extend to support other brokers like central based session store

	  Sample codes for storing data to Memcache:


	  	import memcache


	  	class MySessionStore(guava.session.SessionStore):

			def __init__(self, *args, **kwargs):
				self.conn = memcache.Client(['127.0.0.1:11211'], debug=0)

			def set(self, sid, value):
				return mc.set(sid, value) > 0

			def get(self, sid):
				return mc.get(sid)

			def delete(self, sid):
				return mc.delete(sid) > 0

			def clear(self, sid):
				return mc.set(sid, {}) > 0



	* Cookie Based Session

	  Haven't integrated into guava. I promise will add this feature very soon.


## Super high performance while compared to Flask, web.py...

As you already know, the performace of tornado is very good, but I'm wondering whether my web framework can exceed it.

I chosed C to write the web framework and port to Python as the c extension. If I choose Python to finish the web framework, I'm reinventing the wheel. :)

I got another benifit while following this way which is you can port the guava framework easily to other languages for example: Ruby, PHP... For those who want to port to other languages, please contact me. :)

The builtin web server is building on top of libuv and http-parser. All are released by Joyent(the nodejs company). Libuv is a cross platform async event library, http-parser is for parsing the HTTP Request and Response.

I choose this kind of combination because I can get the best performance.


# Performance Testing

I did a rough performance testing and compared guava with cherrypy, Flask, web.py, tornado.

Maybe the result is not very accurate. Anyways I used the default helloworld for each web framework.

If you guys have more thoughts on this topic, no matter I wrote the bad codes for one specific framework or not did the correct testing, please kindly correct me.


Hardware: My virtual marchine, Ubuntu12.04 2GB Memory 2 cores

Using the ab command for testing with concurrency 50

Command: ```ab -n 40000 -c 50 http://localhost:8000/```

Result:

	| Framework | Result          |
	| Flask     | 1236.89 [#/sec] |
	| CherryPy  | 532.39 [#/sec]  |
	| Tornado   | 2326.46 [#/sec] |
	| Guava     | 5397.37 [#/sec] |


Please keep in mind, the above performance testing result was got from my roughly testing.

In the future, after I finished the stable version of guava, I will do detailed performance testing which will include the transformation of CPU, memory usage and other KPIS.


# Deployment

No WSGI anymore, I will introduce two kinds of tradiontial way for the deployment.


## You can deploy the guava behind of any web servers like Nginx/Apache...


You can configure web servers to proxy all requests to backend Guava server.


            Redirect
	Nginx <-----------> Guava WebServer


Sample configuration for Nginx

	server {
		listen 80;
		server_name mydomain;
		location / {
			proxy_pass http://localhost:8080;
			proxy_redirect off;
			proxy_set_header Host mydomain;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		}
	}



## Use the Guava WebServer as your web server

The performance of the Guava WebServer is good enough for replacing Nginx if your application is not so complex.

But for now, it's not appropriate to choose this kind of deployment, because I haven't spend so much time on the security part. After the basic features of Guava are frozed and stable, I will spend more and more time for optimizing the builtin web server to make it more sophisticated, more powerful.


# For the Future

Currently I'm focused on implementing the basic features like Routers, Session, optimize the performance...


For long-term consideration, I will do following items, probably I will modify this todo list:

* More powerful builtin web server

	I will try to introduce more features for the builtin web server, like integrate with the Cache Control, websocket, better performance

* More unittests for covering the basic features of Guava

* Write more fundation data strutures to remove PyObject related data structures from the core guava codes, so other guys can easily port the guava framework to other languages like Ruby, PHP, Lua... and the core guava codes can become a standalone project.

* Features requested from you guys!


# Contribution

For those guys who wanna contribute to the guava project, I accept all kinds of contributions, including:

1. Contribute to the code codes

2. Write testcases

3. Write the documentation

4. Build the project website

5. Do detailed performance testing

6. Report bugs

7. Describe feature request and send it to me

8. Write sample codes or sample applications

9. Invite developers to evaluate the guava framework

10. More and more...


# Wrap Up

Guava is a super super lightweight high performance web framework for Python written in C.
