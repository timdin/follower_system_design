# Follower system design

The basic requirements are as follows:

Design a system that could handle the following features

- Follow/unfollow another user
- Return and paginate these lists:
  - Following users
  - Followers users
  - Friends users (both following and being followed)

The design should also considered high traffic scenario.

## Assumptions

I'll assume the existing system already have the basic user system, and the schema will be similar to the following chart

| User_id | user_profile_id | ... |
|---------|------|-----|
| system generated id | user defined id | ....|

The above user id should be the identifier for the internal system, while the user profile id will be the identifier user used to retrieve other user profile.

## Basic design

Here is a glimps of how the system handles the user interaction

For query interactions:

```mermaid
sequenceDiagram
user ->> service: request
        activate service
    alt lookup cache success
        service ->> cache: query
        activate cache
        cache ->> service: data exists
        deactivate cache
        service ->> user: return
        deactivate service
    else lookup cache failed
        note over service: not returned yet
        activate service
        service ->> database: query
        database ->> service: return
        service ->> cache: set
        service ->> user: return
        deactivate service
    end
```

For follow/unfollow interactions:

```mermaid
sequenceDiagram
user ->> service: request
activate service
service ->> cache: expire data
service ->> database: update
service ->> user: return
deactivate service
```

Data base design

```mermaid
classDiagram
class user{
    + user_id: int
    + user_profile_id: string
    ...
}

class follows{
    // the user id that follows another user
    + follower_id: int --> user.user_id
    // the user id that being followed
    + followed_id: int --> user.user_id
}
```
