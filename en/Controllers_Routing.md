# Routing

### Default RESTful rules

The main function of router is to connect request URL and handler. Beego wrapped `Controller`, so it connects request URL and `ControllerInterface`. The `ControllerInterface` has following methods:

	type ControllerInterface interface {
		Init(ct *Context, cn string)
		Prepare()
		Get()
		Post()
		Delete()
		Put()
		Head()
		Patch()
		Options()
		Finish()
		Render() error
	}

`beego.Controller` implemented all of them, so you just use this struct as anonymous field in your controller struct. Of course you have to overload corresponding methods for more specific usages.

Users can use following ways to register route rules:

	beego.Router("/", &controllers.MainController{})
	beego.Router("/admin", &admin.UserController{})
	beego.Router("/admin/index", &admin.ArticleController{})
	beego.Router("/admin/addpkg", &admin.AddController{})

For more convenient configure route rules, Beego references the idea from sinatra, so it supports more kinds of route rules as follows:

- beego.Router("/api/:id([0-9]+)", &controllers.RController{})

		Customized regular expression match 	// match /api/123 :id= 123

- beego.Router("/news/:all", &controllers.RController{})

		Match rest of all // match /news/path/to/123.html :all= path/to/123.html

- beego.Router("/user/:username([\w]+)", &controllers.RController{})
 
		Regular expression // match /user/astaxie    :username = astaxie

- beego.Router("/download/`*`.`*`", &controllers.RController{})

		Wildcard character // match /download/file/api.xml     :path= file/api   :ext=xml

- beego.Router("/download/ceshi/`*`", &controllers.RController{})

		wildcard character match rest of all // match  /download/ceshi/file/api.json  :splat=file/api.json

- beego.Router("/:id:int", &controllers.RController{})
 
		Match type int  // match :id is int type, Beego uses regular expression ([0-9]+) automatically

- beego.Router("/:hi:string", &controllers.RController{})

		Match type string // match :hi is string type, Beego uses regular expression ([\w]+) automatically

### Customized methods and RESTful rules

We listed default method name above(methods name are the same as HTTP methods name, like Get for GET requests, Post for POST requests). You may able to customized your method as follows:

	beego.Router("/",&IndexController{},"*:Index")

The 3rd argument indicates the corresponding HTTP method for your method and router rules.

- * means execute this method for all coming requests.
- Use format `<HTTP method>:<function name>` to define routers.
- Use `;` to split multiple pairs.
- If multiple HTTP method are routing to one method, use `,` to split them.

Here is a RESTful style design:

	beego.Router("/api/list",&RestController{},"*:ListFood")
	beego.Router("/api/create",&RestController{},"post:CreateFood")
	beego.Router("/api/update",&RestController{},"put:UpdateFood")
	beego.Router("/api/delete",&RestController{},"delete:DeleteFood")

An example of multiple HTTP method route to one method:

	beego.Router("/api",&RestController{},"get,post:ApiFunc")

Multiple pairs example:

	beego.Router("/simple",&SimpleController{},"get:GetFunc;post:PostFunc")

Available HTTP methods:

- *: For all coming requests.
- get: GET request.
- post: POST request.
- put: PUT request.
- delete: DELETE request.
- patch: PATCH request.
- options: OPTIONS request.
- head: HEAD request.

If you defined * and corresponding HTTP method, Beego chooses HTTP method as prior execution. For example:

	beego.Router("/simple",&SimpleController{},"*:AllFunc;post:PostFunc")

In this example, PostFunc will be executed instead of AllFunc.

### Automated routing

You should register your controllers for auto-routing:

	beego.AutoRouter(&controllers.ObjectController{})

Then Beego reflects all methods in `ObjectController` and register corresponding routers:

	/object/login   Call Login method of ObjectController.
	/object/logout  Call Logout method of ObjectController.

In addition to matching of two prefixes:`/:controller/:method`, Beego automation resolved rest of URL as parameters, and save into this. Ctx. Params:

	/object/blog/2013/09/12  Call Blog method of ObjectController, and has the argument map[0:2013 1:09 2:12].

The URL will be converted lower case before matching; therefore, `/object/LOGIN` routes to Login method as well.
