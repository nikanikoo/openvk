# VK API Compatability layer for OpenVK

This directory contains VK API handlers, structures and relared
exceptions. It is still a work-in-progress functionality.  
**Note**: requests to API are routed through
openvk.Web.Presenters.VKAPIPresenter, this dir contains only handlers.

[Documentation for API clients](https://openvk.org/dev)

## Implementing API methods

VK API methods have names like this: `example.test`. To implement a
method like this you will need to create a class `Example` in the
Handlers subdirectory. This class **must** extend VKAPIHandler and be
final.  
Next step is to create test method. It **must** have a type hint that is
not void. Everything else is fine, the return value of method will be
authomatically converted to JSON and sent back to client.

### Parameters

Method arguments are parameters. To declare a parameter just create an
argument with the same name. You should also provide correct type hints
for them. Type conversion is done automatically if possible. If not
possible error №1 will be returned.  
If parameter is not passed by client then router will pass default value
to argument. If there is no default value but argument accepts NULL then
NULL will be passed. If NULL is not acceptable, default value is
undefined and parameter is not passed, API will return missing parameter
error to client.

### Returning errors

To return an error, call fail method like this: `$this->fail(5,
"error")` (first argument is error code and second is error message).
You can also throw the exception manually: `throw new
APIErrorException("error", 5)` (class:
openvk.VKAPI.Exceptions.APIErrorException).  
If you throw any exception that does not inherit APIErrorException then
API will return error №1 (unknown error) to client.

### Refering to user

To get user use `getUser` method: `$this->getUser()`. Keep in mind it
will return NULL if user is undefined (no access\_token passed or it is
invalid/expired or roaming authentification failed).  
If you need to check whether user is defined use `userAuthorized`. This
method returns true if user is present and false if not.  
If your method can’t work without user context call `requireUser` and it
will automatically return unauthorized error.

### Working with data

You can use OpenVK models for that. However, **do not** return them
(either you will leak data or JSON conversion will fail). It is better
to create a response object and return it. It is also a good idea to
define a structure in Structures subdirectory.

## The `execute` method (VKScript)

OpenVK provides full support for the VK API `execute` method and VKScript language interpreter.

* **Endpoint:** `/method/execute` or `/method/execute.<procedure>`
* **Parameter:** `code` (string) — the VKScript code to execute.
* **Returns:** `{"response": ...}` and optional `{"execute_errors": [...]}`.

### VKScript Features:
- **Variables & Arguments:** `var x = 1;` and access to query parameters via `Args.paramName`.
- **API calls:** `API.users.get({user_ids: 1})` or `API.wall.get(...)` (up to 25 calls per script).
- **Operators:** Arithmetic (`+`, `-`, `*`, `/`, `%`), bitwise (`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`), comparisons (`==`, `!=`, `===`, `!==`, `<`, `>`, `<=`, `>=`), compound assignments (`+=`, `-=`, `*=`, `/=`, etc.), increment/decrement (`++`, `--`), ternary (`cond ? a : b`), `typeof`, `delete`.
- **Array projection:** `items@.id` or `items@[field]`.
- **Control flow:** `if/else`, `while`, `do..while`, `for (var i = 0; i < n; i++)`, `break`, `continue`, `return`.
- **Built-in Objects & Methods:**
  - `Math` (`min`, `max`, `floor`, `ceil`, `round`, `abs`, `trunc`, `pow`, `sqrt`, `random`, `sin`, `cos`, `tan`, `atan2`, `log`, `exp`, `PI`, `E`)
  - `JSON` (`parse`, `stringify`)
  - `Object` (`keys`, `values`, `entries`, `assign`)
  - `Array` (`isArray`, `push`, `pop`, `shift`, `unshift`, `slice`, `splice`, `indexOf`, `lastIndexOf`, `reverse`, `join`, `concat`, `includes`)
  - `String` (`substr`, `substring`, `slice`, `split`, `indexOf`, `lastIndexOf`, `includes`, `startsWith`, `endsWith`, `toLowerCase`, `toUpperCase`, `trim`, `charAt`, `charCodeAt`, `replace`, `repeat`, `padStart`, `padEnd`)
  - Global helpers: `parseInt`, `parseFloat`, `isNaN`, `isFinite`, `encodeURIComponent`, `decodeURIComponent`, `escape`, `unescape`, `String`, `Number`, `Boolean`.

Have a lot of fun <sup></sup>

