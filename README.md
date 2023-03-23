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
