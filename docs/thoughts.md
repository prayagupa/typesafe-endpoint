I want to build Spring REST API that can generate service method and repository at compile automatically.

Be creative in naming. 
Example

```
class UserController {
    @ApiGetMetadata(
        entity = UserEntity.class,
        lookupBy = UserEntity.id
        alsoGet = {user.order}
    )
    public UserDetailResponseDto getUser(...);

}
```
