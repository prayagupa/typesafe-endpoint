I want to build Spring REST API that can generate service method and repository at compile automatically.

Be creative in naming. 
Example

```
//hand written
class UserEntity {

}

class UserDetailResponseDto {


}

class UserController {

    @ApiGetMetadata(
        entity = UserEntity.class,
        lookupBy = UserEntity.id,
        alsoGet = {user.order}
    )
    public UserDetailResponseDto getUser(...);

}
```

- include pagination

Gotchas:
- For one to many relations, i should define separate typesafe Response DTO. Meaning if user asks just for 
/users/id, api should respond UserResponseDto, but with /users/id?trail=user.order then UserDetailResponseDto

- same applies for ManyToOne