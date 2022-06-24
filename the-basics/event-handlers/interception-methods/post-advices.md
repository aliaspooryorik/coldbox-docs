# Post Advices

![](<../../../.gitbook/assets/eventhandler-prepost (2) (2) (2) (1) (2) (1).jpg>)

With this interceptor you can intercept local event actions and execute things **after** the requested action executes. You can do it globally by using the `postHandler()` method or targeted to a specific action `post{actionName}()`.

```javascript
// executes after any action
function postHandler( event, rc, prc, action, eventArguments ){
}

// executes after the list() action ONLY
function postList( event, rc, prc, action, eventArguments ){
}

// concrete examples
function postHandler( event, rc, prc, action, eventArguments ){
    log.info("Finalized executing #action#");
}
```

The arguments received by these interceptors are:

* `event` : The request context reference
* `action` : The action name that was intercepted
* `eventArguments` : The struct of extra arguments sent to an action if executed via `runEvent()`
* `rc` : The **RC** reference
* `prc` : The **PRC** Reference

## Exceptions & Only Lists

You can fine tune these interception methods by leveraging two public properties in the handler:

* `this.posthandler_only` : A list of actions that the `postHandler()` action will fire ONLY!
* `this.posthandler_except` : A list of actions that the `postHandler()` action will NOT fire on

```javascript
// only fire for the actions: save(), delete()
this.posthandler_only = "save,delete";
// DO NOT fire for the actions: login(), doLogin(), logout()
this.posthandler_except = "login,doLogin,logout"
```
