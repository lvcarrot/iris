# Controllers from scratch

This folder shows how [@kataras](https://github.com/kataras) started to develop
the MVC idea inside the Iris web framework itself.

**Now** it has been enhanced and it's a **built'n** feature and can be used as:

```go
package main

import (
	"sync"

	"github.com/kataras/iris"
	"github.com/kataras/iris/mvc"
)

func main() {
	app := iris.New()
	app.RegisterView(iris.HTML("./views", ".html"))

	// when we have a path separated by spaces
	// then the Controller is registered to all of them one by one.
	//
	// myDB is binded to the controller's `*DB` field: use only structs and pointers.
	app.Controller("/profile /profile/browse /profile/{id:int} /profile/me",
		new(ProfileController), myDB) // IMPORTANT

	app.Run(iris.Addr(":8080"))
}

// UserModel our example model which will render on the template.
type UserModel struct {
	ID       int64
	Username string
}

// DB is our example database.
type DB struct {
	usersTable map[int64]UserModel
	mu         sync.RWMutex
}

// GetUserByID imaginary database lookup based on user id.
func (db *DB) GetUserByID(id int64) (u UserModel, found bool) {
	db.mu.RLock()
	u, found = db.usersTable[id]
	db.mu.RUnlock()
	return
}

var myDB = &DB{
	usersTable: map[int64]UserModel{
		1:  {1, "kataras"},
		2:  {2, "makis"},
		42: {42, "jdoe"},
	},
}

// ProfileController our example user controller which controls
// the paths of "/profile" "/profile/{id:int}" and "/profile/me".
type ProfileController struct {
	mvc.Controller // IMPORTANT

	User UserModel `iris:"model"`
	// we will bind it but you can also tag it with`iris:"persistence"`
	// and init the controller with manual &PorifleController{DB: myDB}.
	DB *DB
}

// Get method handles all "GET" HTTP Method requests of the controller's paths.
func (pc *ProfileController) Get() { // IMPORTANT
	path := pc.Path

	// requested: /profile path
	if path == "/profile" {
		pc.Tmpl = "profile/index.html"
		return
	}
	// requested: /profile/browse
	// this exists only to proof the concept of changing the path:
	// it will result to a redirection.
	if path == "/profile/browse" {
		pc.Path = "/profile"
		return
	}

	// requested: /profile/me path
	if path == "/profile/me" {
		pc.Tmpl = "profile/me.html"
		return
	}

	// requested: /profile/$ID
	id, _ := pc.Params.GetInt64("id")

	user, found := pc.DB.GetUserByID(id)
	if !found {
		pc.Status = iris.StatusNotFound
		pc.Tmpl = "profile/notfound.html"
		pc.Data["ID"] = id
		return
	}

	pc.Tmpl = "profile/profile.html"
	pc.User = user
}

/* Can use more than one, the factory will make sure
that the correct http methods are being registered for each route
for this controller, uncomment these if you want:

func (pc *ProfileController) Post() {}
func (pc *ProfileController) Put() {}
func (pc *ProfileController) Delete() {}
func (pc *ProfileController) Connect() {}
func (pc *ProfileController) Head() {}
func (pc *ProfileController) Patch() {}
func (pc *ProfileController) Options() {}
func (pc *ProfileController) Trace() {}
*/

/*
func (c *ProfileController) All() {}
//        OR
func (c *ProfileController) Any() {}
*/
```

Example can be found at: [_examples/mvc](https://github.com/kataras/iris/tree/master/_examples/mvc).