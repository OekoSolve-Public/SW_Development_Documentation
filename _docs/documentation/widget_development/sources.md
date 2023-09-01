---
layout: default
title: Sources
parent: ThingsBoard Widget Development
grand_parent: Documentation
nav_order: 6
---

## Sources

[ThingsBoard Widget Development Guide](https://thingsboard.io/docs/user-guide/contribution/widgets-development/#introduction) - Includes some basic information on the UI and layout, some basic example and the basic widget `API`, but more advanced `API` calls are not included and links to where to find them are not included as well

[ThingsBoard GitHub Repository](https://github.com/thingsboard/thingsboard/blob/master/) - Includes the code to all services and their implementation, allows to see which calls can be done from Widgets and which service has to be used to call that method

[ThingsBoard Rest API](https://demo.thingsboard.io/swagger-ui/) - Includes the complete `Rest API` with documentation on all `POST`, `GET`, `PUT` and `DELETE` requests that can possibly be called by `ThingsBoard` with the needed user access right, a detailed description on parameters and an example request and response `json`, does not mean they can be called from Widgets, because to do that a service needs to wrap that call first, if that is the case that method should be found in the [ThingsBoard GitHub Repository Services](https://github.com/thingsboard/thingsboard/blob/master/ui-ngx/src/app/core/http/)

[ThingsBoard GitHub Repository Issues](https://github.com/thingsboard/thingsboard/issues) - Includes problems of other people that might have tried to implement the same feature or utility with a `widget`

[ThingsBoard PE Widget Library](https://thingsboard.io/docs/pe/user-guide/ui/widget-library/) - Includes some basic information about the different `widget` types

[ThingsBoard PE Advanced data key configuration](https://thingsboard.io/docs/pe/user-guide/ui/advanced-data-key-configuration/) - Includes some basic information on how the value of keys can be configured and adjusted before it is displayed on the [`Dashboard`](https://thingsboard.io/docs/user-guide/dashboards/)
