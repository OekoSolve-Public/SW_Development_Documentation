---
layout: default
title: Creating a Widget
parent: Widget Development
grand_parent: ThingsBoard
nav_order: 5
grand_grand_parent: Documentation
---

## Creating a Widget

Now that the basics have been shown and can be looked up if ever required again this is a step by step guide on how to now actually create a widget.

1. **What does the widget need to do?**: Note down what exactly the widget needs to read from or write to 
2. **What type of objects does it need access to?**: Note down what type of objects would give the required information that should be read, or what objects need to be written too
3. **Search for [`Rest API`](https://<TB_PE_PAGE_LINK>/swagger-ui/#/) calls**: Search for API calls that fufill thoose requirements, note down the `controller name`, `api url`, `controller method name`
4. **Search for the method with the given `api url` in the GitHub repository [folder](https://github.com/thingsboard/thingsboard/blob/master/ui-ngx/src/app/core/http/)**: Most of the cases the `api url`, should lead to an implemented method that calls that api (only search for the first part of the `api url` until variables, for example `api/customer/`). If the implemented method could be found note the `service method name` and the name of the `file` the method is implemented in that `file name` is the `service name`. The method should now be able to be called on any widget with the format of `self.ctx.<service name>.<service method name>`. The `Basic Widget API`, `Services` section contains more information.

[!WARNING]
If no implemented method could be found check the [`ThingsBoard Community Rest API`](https://<TB_PE_PAGE_LINK>/swagger-ui/#/), that contains only calls that exist for `ThingsBoard Community Edition`. Because if your call exists only on `ThingsBoard Professional Edition`, you will not find it in the GitHub repository [folder](https://github.com/thingsboard/thingsboard/blob/master/ui-ngx/src/app/core/http/). If that is the case  the `Basic Widget API`, `Services`, `Using ThingsBoard Professional Edition only Widget features` section contains more information on how this can be mitigated.
