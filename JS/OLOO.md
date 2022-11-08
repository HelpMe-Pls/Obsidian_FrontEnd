- Using the OLOO (Objects Linking to Other Objects) pattern to optimize the benefits of the "linking" that the "prototype" system offers (also make your code more "testable"):

```javascript
var Workshop = {
    setTeacher(teacher) {
        this.teacher = teacher
    },

    ask(question) {
        console.log(this.teacher, question)
    }
}

  

var AnotherWorkshop = Object.assign(
    Object.create(Workshop),
    {
        speakUp(msg) {
            this.ask(msg.toUpperCase())
        }
    }
)


var JSRecentParts = Object.create(AnotherWorkshop)

JSRecentParts.setTeacher("K")

JSRecentParts.speakUp("this is how it should be")

// "K THIS IS HOW IT SHOULD BE"
```



- The so called "Inheritance" in other Object-oriented languages is actually "delegation" in JS. Inheritance is understood as a "parent-to-child" relationship where "**delegation**" more like a "**peer-to-peer**" relationship:

```javascript

var AuthController = {
    authenticate() {
        console.log("auth");
    },
    
    handleResponse() {
        this.displayError();
    }
};

  

var LoginFormController = Object.assign(
  Object.create(AuthController),
  {
      onSubmit() {
        this.authenticate(); // "delegated" to `AuthController`
      },

      displayError() {
        console.log("login");
      }
  }
);

  

var mockAuth = Object.create(AuthController);

var mockLogin = Object.create(LoginFormController);


// wrong:

// mockAuth.onSubmit()

// mockAuth.displayError()

// mockAuth.handleResponse();

  

// correct:

mockLogin.handleResponse();

mockLogin.onSubmit();

mockAuth.authenticate();
```