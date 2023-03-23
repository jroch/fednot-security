# Roles and abilities

Whenever Trustbuilder and API connect consume our application / API, they are injecting common security headers, including the `iv-groups` header.
The groups (or _roles_) that are being passed in should be translated into abilities: _what can this user do_?

The abilities are mapped in a table documented on https://confluence.fednot.be/display/PROJ/01.+Service+Authorize+Access:

| Role                       | Ability                 |
|----------------------------|-------------------------|
| grp_crf_registration       | registrations.index     |
| grp_crf_registration       | registrations.create.   |
| ...                        | ...                     |

This table is currently maintained in the code at this location: https://bitbucket.fednot.be/projects/REG/repos/backend/browse/crf-registers-domain/src/main/resources/ResourceAccessControl.json
So whenever we expect a new user group to authenticated that we want to authorize access and to perform specific actions, we should add the group to all _actions_ that it needs to be able to do.

# Validating abilities on the backend

When we need to validate whether a user has access to perform a specific action, we should NOT do

```
if (user belongs to group XXX) {
  // do something
}
```

but instead should always check whether the authenticated user has the _ability_ to do something:

```
if (user can XXX) {
  // do something
}
```

This has been implemented in the backend by the following annotation calling the `hasPermission` functionality:

```java 
@PreAuthorize("hasPermission(null, T(be.fednot.registers.deed.access.Permission).CREATE_CONSULTATION)")
```

# Validating abilities on the frontend

*This is a proposal to fix the issues we currently have on the frontend*

## Getting a list of abilities or permissions on the frontend.

We currently have the list of abilities managed on the backend.  My suggestion would be to extend the GET /context endpoint and add an authorization attribute that has a list of permissions allowed:

```json
{
  participantType: { ... },
  version: { ... },
  authentication: { ... },
  authorization: {
    permissions: [
      'registrations.index',
      'registrations.created',
      ...
    ]
  }
}
```

On the backend, you would basically just have to fetch the permissions from the config file for the list of `iv-groups` in the request headers.

## Client implementation

If we can just implement a nice little composable helper on the frontend, we're right on track:

### store the permissions returned from the GET /context call

Upon loading the page, we are already calling GET /context - upon doing that, we shoud store the returned context.authorizations.permissions into a client side store.

### Validating access to features

On the client side, whenever we need to validate access to certain features, we need to check whether the authenticated user has the ability to do something.  This can easily be done by defining a composable that basically does something like this:

```js

let function can = (permission) => {
  return $authorizationStore.permissions.includes(permission);
}
```

Next step would be to just replace anything where we are checking for permissions by a call to `can('registrations.index')`
 which would be a simple yet easy and performant solution to the problem.
  
  
 


