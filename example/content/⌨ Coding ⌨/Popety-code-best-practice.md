### Introduction

> “Always code as if the guy who ends up maintaining your code will be a violent psychopath who knows where you live”
― John Woods

##### Javascript is moving from ES5 to ES6. So, we will try to follow ES6 coding structure and definitions as much as possible.

```
MORE ABOUT ES6: http://es6-features.org/
```

### Structure

#### /api
  > _/api_ contains all the api endpoint functions.
  * api.js contains all api end points. If we are going to start the development of the new api, we need to start from this file. Continue with following steps to define api end points,
      * First check if the api endpoint is public, private or admin. (_currently we have three types of user access_).

      ```
        For
          1. Admin apis use "adminAuth" interceptor.
          2. User related private apis use "auth" interceptor.
          3. Public apis should have the "publicAuth" interceptor.

        Example:

          app.put('/api/v2/admin/services/provider', adminAuth, admin.addServiceProvider); // Admin Apis

          app.get('/api/v2/defects/types', auth, defect.getTypes); // User Private Apis

          app.get('/api/v2/propertyTypes', publicAuth, property.getTypes); // Public Apis
      ```

  <!-- * certificates
  * common
  * config
  * db
  * dr
  * event
  * fonts
  * interceptors
  * job
  * migration
  * process
  * quickFixJob
  * socket
  * sql
  * test -->
