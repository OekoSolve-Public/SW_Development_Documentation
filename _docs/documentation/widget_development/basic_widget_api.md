---
layout: default
title: Basic Widget API
parent: ThingsBoard Widget Development
grand_parent: Documentation
nav_order: 3
---

## Basic Widget API

This section contains basic information about widgets that is shared by every different widget type. See [Basic Widget API](https://thingsboard.io/docs/user-guide/contribution/widgets-development/#basic-widget-api) for more information on the methods below. As well as the development `UI` for widgets.

### [Widget Editor overview](https://thingsboard.io/docs/user-guide/contribution/widgets-development/#widget-editor-overview)

The widget editor contains of five main section.

- Widget Editor Toolbar
- Resources / `HTML` / `CSS`
- `JavaScript`
- `Settings schema`
- Widget preview

![Widget Editor](images/image-11.png)

#### Widget Editor Toolbar section

The widget editor toolbar consists of the following items in the order from left to right.

- Widget Title field (Used to specify widget title, useful to know what the widget does)
- Widget Type selector (Used to specify widget type)
- Run button (Used to run widget code and view result in Widget preview section)
- Undo button (Reverts all editor sections to latest saved state)
- Save button (Saves current widget changes)
- Save as button (Allows to save a new copy of the widget by specifying new widget name and target `Widgets Bundle`)

#### Resources / `HTML` / `CSS` section

The first tab `Resources` is used to define external `JavaScript` or `CSS` dependencies used by the widget

![Resources](images/image-12.png)

The second tab `HTML` contains the widget `HTML` code, that decides how the widget is displayed. Can be empty, if the content is created dynamically.

![HTML](images/image-13.png)

The last tab in this section is the `CSS` tab, that decies how the `HTML` code should be styled.

![CSS](images/image-14.png)

#### `JavaScript` section

This section contains the `JavaScript` code that should be executed when a user interacts with the widget, everything mentioned in the `Basic Widget API`, `API` section can be done here.

#### Settings schema section

The first tab `Settings schema` is used to specify the displayed settings for the `Advanced` tab of the widget settings.

An example would be a simple `Settings schema` that displays only one user input box for text.

```json
{
    "schema": {
        "type": "object",
        "title": "Settings",
        "properties": {
            "example": {
                "title": "Example user input in Widget Advanced section",
                "type": "string",
                "default": "Test"
            },
            "required": []
        }
    },
    "form": [
        "example"
    ]
}
```

This configuration can be easily read and accessed in the `JavaScript` code of the widget. With the format of `self.ctx.settings.<properties-name>`.

```js
function init() {
    var example_value = self.ctx.settings.example;
}
```

To know what to put into the `Settings schema`, the easiest way is to use the [React Schema Form](https://networknt.github.io/react-schema-form/) to create and display the needed inputs and then simply copy the `Form` and `Schema` sections and paste them into the `"schema"` and `"form"` sections in the widget input.

The second tab `Data key settings schema` is used to specify the displayed settings for the `Advanced` tab of the data key settings. The `.json` format is the same as for `Settings schema`.

![Data key advanced](images/image-15.png)

Getting the configuration however is slightly different, because the configuration can be different for every single `data key` that is given by the entered `data source`.

```js
function init() {
    for (var datasource in self.ctx.datasources) {
        for (var data_key in datasource.dataKeys) {
            var example_value = data_key.settings.example;
        }
    }
}
```

#### Widget preview section

This section allows to preview how to widget would be displayed on the [`Dashboard`](https://thingsboard.io/docs/user-guide/dashboards/). The only limitation is that the only `data source` that can be selected is a `Function`.

![Preview](images/image-16.png)

### API

The base functions created for every widget.

```js
self.onInit = function() {
    // First function called when widget is ready for initalization.
    self.ctx.ngZone.run(function() {
       init(); 
       self.ctx.detectChanges(true);
    });
}

function init() {
    // Should be used to process settings and get needed services
}

self.onDataUpdated = function() {
    // Called when new data of the data source is available
}

self.onResize = function() {
    // Called when widget is resized
}

self.onEditModeChanged = function() {
    // Called when dashboard editing mode is left or entered
}

self.onDestroy = function() {
    // Called when widget is not shown anymore
}
```

### Services

To let widgets handle more complex use cases, like interacting with data and updating or even deleting it, it needs to interact with the `Rest API`.

This can not be done directly, the interaction is handled over `services` that expose specific `Rest API` calls via custom methods on those `services`.

To see all possible `Rest API` calls for the client [Swagger UI](https://<TB_PE_PAGE_LINK>/swagger-ui) can be accessed. Most of the `controller` mentioned can be found as `<controller-name>.service.ts` in the GitHub repository under the [ui-ngx/src/app/core/http](https://github.com/thingsboard/thingsboard/blob/master/ui-ngx/src/app/core/http) folder.

[!WARNING]
Be aware tough, some calls might only be included in the `ThingsBoard Professional Edition`, if that is the case the implementation will not be visible in the GitHub repository [ui-ngx/src/app/core/http](https://github.com/thingsboard/thingsboard/blob/master/ui-ngx/src/app/core/http) folder. If that is the case  the `Basic Widget API`, `Services`, `Using ThingsBoard Professional Edition only Widget features` section contains more information on how this can be mitigated.

When having a closer look at the method implementation the specific `Rest API` call used can be deduced by the `URL`.

```ts
public getDevice(deviceId: string, config?: RequestConfig): Observable<Device> {
    // GET /api/device/${deviceId}
    // https://<TB_PE_PAGE_LINK>/swagger-ui#/device-controller/getDeviceByIdUsingGET
    return this.http.get<Device>(`/api/device/${deviceId}`, defaultHttpOptionsFromConfig(config));
}
```

Once the `Rest API` call has been deduced this then helps to deduce which users this call is even available too. Being either:

- `TENANT_ADMIN` (Administrators of the current tenant instance)
- `CUSTOMER_USER` (Users of customers of the current tenant instance)

Additional it is written, what kind of permission is required to execute the call on the given [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/). The `operations` can be found in the GitHub repository under the [permissions/Operation.java](https://github.com/thingsboard/thingsboard/blob/master/application/src/main/java/org/thingsboard/server/service/security/permission/Operation.java#L21) section.

#### Using ThingsBoard Professional Edition only Widget features

Some `services` only exist in the `ThingsBoard Professional Edition`, because their `Rest API` calls only exist in the `ThingsBoard Professional Edition`.

If that is the case the only way to find out if the `controller` is implemented is trying to guess the `service` name from the name it has in the `Rest API`.

Additional auto complete in the `Widget Editor` is also borked for thoose `ThingsBoard Professional Edition` only `services` so the methods need to be found out another way.

This is possible with the below code, simply enter the name of the `service` class, which we want to call the methods on. Once executed in the `Widget` it will print all existing `methods` on that object. Original code can be found here [How to list object methods javascript](https://flaviocopes.com/how-to-list-object-methods-javascript).

```javascript
console.log(getMethods("<TB_PE_ONLY_WIDGET_SERVICE_NAME>"))

const getMethods = (obj) => {
  let properties = new Set();
  let currentObj = obj;
  do {
    Object.getOwnPropertyNames(currentObj).map(item => properties.add(item));
  } while ((currentObj = Object.getPrototypeOf(currentObj)) != undefined);
  return [...properties.keys()].filter(item => typeof obj[item] === 'function');
};
```

```json
[
    "constructor",
    "getOwners",
    "getEntityGroup",
    "fetchOtaPackageGroupInfo",
    "updateDeviceGroupOtaPackage",
    "saveDeviceGroupOtaPackage",
    "saveDeviceEntityGroup",
    "deleteDeviceGroupOtaPackage",
    "saveEntityGroup",
    "deleteEntityGroup",
    "makeEntityGroupPublic",
    "makeEntityGroupPrivate",
    "shareEntityGroup",
    "getEntityGroups",
    "getEntityGroupIdsForEntityId",
    "getEntityGroupsByIds",
    "getEntityGroupsByOwnerId",
    "getEntityGroupsByOwnerIdAndPageLink",
    "getEntityGroupAllByOwnerId",
    "getEntityGroupsByPageLink",
    "addEntityToEntityGroup",
    "addEntitiesToEntityGroup",
    "changeEntityOwner",
    "removeEntityFromEntityGroup",
    "removeEntitiesFromEntityGroup",
    "getEntityGroupEntity",
    "getEntityGroupEntities",
    "getEdgeEntityGroups",
    "assignEntityGroupToEdge",
    "unassignEntityGroupFromEdge",
    "__defineGetter__",
    "__defineSetter__",
    "hasOwnProperty",
    "__lookupGetter__",
    "__lookupSetter__",
    "isPrototypeOf",
    "propertyIsEnumerable",
    "toString",
    "valueOf",
    "toLocaleString"
]
```

These `method` names can now be used to discern the correct `Rest API` call, because most of the time the `method` names are relatively similair.

```javascript
entityGroupController.getEntityGroupAllByOwnerId();
// Rest API: https://<TB_PE_PAGE_LINK>/swagger-ui/#/entity-group-controller/getEntityGroupAllByOwnerAndTypeUsingGET
```

Once the correct method has been deduced the `parameters` needed to be passed to the `Rest API` can be used, normally the order and the argument the `service` method accepts are exactly the same.

If the `Rest API` call additional expects a `request body` that will simply the last element as an optional.

-------------------

The important thing is to remember that those access right do not mean that those actions can be executed on every device. The executing user still has to have access to the given [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/), meaning he has to have created it, it has to be assigned to him or it has to have been shared with him.

- `ALL` (Allows for all action mentioned below this one)
- `CREATE` (Allows to create a new [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/))
- `READ` (Allows to access an [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) and read its information)
- `WRITE` (Allows to write to an [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/))
- `DELETE` (Allows to delete an [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/))
- `ASSIGN_TO_CUSTOMER` (Allows to assign an [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) to a customer allowing that customer to access that [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) with the access rights assigned to that customer depends on the assigned Role)
- `UNASSIGN_FROM_CUSTOMER` (Allows to unassign an  [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) from a customer)
- `RPC_CALL` (Allows to execute a [`Remote Procedure Call`](https://thingsboard.io/docs/user-guide/rpc/) on a given device)
- `READ_CREDENTIALS` (Allows to read the credentials of a given device)
- `WRITE_CREDENTIALS` (Allows to update or create the credentials on a given device)
- `READ_ATTRIBUTES` (Allows to read attributes of a given device)
- `WRITE_ATTRIBUTES` (Allows to update or create attributes on a given device)
- `READ_TELEMETRY` (Allows to read telemetry of a given device)
- `WRITE_TELEMETRY` (Allows to update or create telemtry on a given device)
- `CLAIM_DEVICES` (Allows to claim devices as their own, automatically assigning it to the person claiming)
- `ASSIGN_TO_TENANT` (Allows to assign device to a tenant)

The specific roles mentioned above might ring a bell, especially if you previously worked with `ThingsBoard Roles`. Because nearly all permissions mentioned above can be added to a `Role` that can then be assigned to a [`User`](https://thingsboard.io/docs/user-guide/ui/users/). This is especially helpful if the `Rest API` is called but failed, because the [`User`](https://thingsboard.io/docs/user-guide/ui/users/) does not have the correct permissions to execute that `Rest API` call.

![Roles](images/image-9.png)

-------------------

The specific credentials required authorization and `HTTPS` call type of each method in the services can be found as `<controller-name>Controller.java` in the GitHub repository under the [application/src/main/java/org/thingsboad/server/controller](https://github.com/thingsboard/thingsboard/tree/master/application/src/main/java/org/thingsboard/server/controller) folder.

```java
@ApiOperation(value = "Get Device (getDeviceById)",
              notes = "Fetch the Device object based on the provided Device Id. " +
                      "If the user has the authority of 'TENANT_ADMIN', the server checks that the device is owned by the same tenant. " +
                      "If the user has the authority of 'CUSTOMER_USER', the server checks that the device is assigned to the same customer." +
                    TENANT_OR_CUSTOMER_AUTHORITY_PARAGRAPH)
@PreAuthorize("hasAnyAuthority('TENANT_ADMIN', 'CUSTOMER_USER')")
@RequestMapping(value = "/device/{deviceId}", method = RequestMethod.GET)
@ResponseBody
public Device getDeviceById(@ApiParam(value = DEVICE_ID_PARAM_DESCRIPTION)
                            @PathVariable(DEVICE_ID) String strDeviceId) throws ThingsboardException {
    checkParameter(DEVICE_ID, strDeviceId);
    DeviceId deviceId = new DeviceId(toUUID(strDeviceId));
    return checkDeviceId(deviceId, Operation.READ);
}
```

Especially important are the `@PreAuthorize`, `@RequestMapping` and `Operation.<PERMISSION>` section., which describe the type of user that is allowed to execute the action, the `URL` the `Rest API` call is mapped too and the operation permission to execute that action.

-------------------

Now that we have had a closer look at the permission required to execute methods on services and where to find what kind of calls we can even execute. The next things is to actually implement them.

For that we can take an already existing widget or create a new one.
The most important section is to get the needed services in the `init` function. This is done to ensure the servies are loaded only once.

```js
let deviceService;

self.onInit = function() {
    self.ctx.ngZone.run(function() {
       init(); 
       self.ctx.detectChanges(true);
    });
}

function init() {
    var $scope = self.ctx.$scope;
    var $injector = $scope.$injector;
    // Services that allow access to preprogrammed methods, that forward the given variables,
    // to the REST API and give back the wanted data if the user has the correct access rights
    deviceService = self.ctx.deviceService;
}
```

We can now execute any of the previously found methods in the [`device.service.ts`](https://github.com/thingsboard/thingsboard/blob/master/ui-ngx/src/app/core/http/device.service.ts). Be aware tough `ThingsBoard` uses `Angular` under the hood meaning we do not block and then receive the result directly from the method, but instead work with a kind of promise.

Meaning we can `subscribe` a `success` and `failure` method and those will be called once the method has `failed` or `succeded` to execute the given `Rest API` call.

This means that directly returning the call can not be done, we always have to wait until we get the response and then change for example the value of a `UI` component or log the received data into the console.

```js
var device_id = "ID_OF_THE_DEVICE";
deviceService.getDevice(device_id).subscribe(
    (data) => {
        console.log(data);
        // Write received data that should be displayed into a HTML element so it is displayed.
        document.getElementById('deviceJson').innerHTML = data;
    },
    (error) => {
        // Display error toast to inform the user of a failed request.
        $scope.showErrorToast("Device not found", 'bottom', 'left', $scope.toastTargetId);
    }
);
```
