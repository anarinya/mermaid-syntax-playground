# Class Diagram

### Use Cases
* Larger refactors, and what methods and dependencies are at play

### Class Diagram vs Regular Domain Diagram
They both use mermaid `classDiagram`.
It's ideal to define classes separately, before relationships.
* Visibility of a property is identified with private `-` or public `+`
* Dependencies are identified with arrows, such as `..>`

```mermaid
classDiagram
  class UserController {
    -CreateUserService _createUserService
    +UserController(CreateUserSerice createUserSerice)
    +Create(string email, string username) User
  }
  class CreateUserService {
    -UserModel _userModel
    -SendWelcomeEmailService _sendWelcomeEmailService
    -Kafka _kafka
    +CreateUserService(UserModel um, SendWelcomeEmailService es, Kafka k)
    +Call(string email, string username) User
    -CheckActiveUsers(List~User~ users) bool
  }
  class UserModel {
    +FindUsersByEmail(string email) List~User~
    +CreateUser(string email, string username) User
  }
  class User {
    +int UserId
    +string Username
    +string Email
  }
  class SendWelcomeEmailService {
    +Call(string email, string username) bool
  }
  class Kafka {
    +PublishUserCreatedEvent(string email, string username) bool
  }
  UserController ..> CreateUserService: depends on
  CreateUserService ..> UserModel: depends on
  UserModel ..> User: depends on
  CreateUserService ..> SendWelcomeEmailService: depends on
  CreateUserService ..> Kafka: depends on

  UserController ..> CreateUserService: depends on
```

### Refactoring
*Identify code smells:*
Several classes contain the same parameters, but each time we're passing parameters individually as primitive types. If we had more parameters, the code smell would be `long parameter list`, but since there are 2, it's a `data clump`. This code smell can be identified by having the same pieces of data passed together, but as separate parameters.

To fix, create a class to group data points and pass that in the method signature instead (instance of User).

Another code smell is that concrete implementations of classes are relied on, instead of using interfaces, which would use no logic and just define methods that a class must implement to fill a contract.

**Example**
```
class ICreateUserService {
  <<Interface>>
  +Call(CreateUserRequest createUserRequest) User
}
```

### Final Diagram

```mermaid
classDiagram
  class ICreateUserService {
    <<Interface>>
    +Call(CreateUserRequest createUserRequest) User
  }
  class UserController {
    <<Class>>
    -ICreateUserService _createUserService
    +UserController(ICreateUserService createUserService)
    +Create(string email, string username) User
  }
  class CreateUserRequest {
    <<Class>>
    +string Email
    +string Username
    +Validate() bool
  }
  class CreateUserService {
    <<Class>>
    -UserModel _userModel
    -SendWelcomeEmailService _sendWelcomeEmailService
    -Kafka _kafka
    +CreateUserService(UserModel um, SendWelcomeEmailService es, Kafka k)
    +Call(CreateUserRequest createUserRequest) User
    -CheckActiveUsers(List~User~ users) bool
  }
  class UserModel {
    <<Class>>
    +FindUsersByEmail(string email) List~User~
    +CreateUser(string email, string username) User
  }
  class User {
    <<Class>>
    +int UserId
    +string Username
    +string Email
  }
  class SendWelcomeEmailService {
    <<Class>>
    +Call(User user) bool
  }
  class Kafka {
    <<Class>>
    +PublishUserCreatedEvent(User User) bool
  }
  UserController ..> ICreateUserService: depends on
  UserController ..> CreateUserRequest: depends on
  CreateUserService ..|> ICreateUserService: implements
  CreateUserService ..> UserModel: depends on
  UserModel ..> User: depends on
  CreateUserService ..> SendWelcomeEmailService: depends on
  CreateUserService ..> Kafka: depends on
```
