---
layout: default
title: Widget Types
parent: Widget Development
grand_parent: ThingsBoard
nav_order: 2
grand_grand_parent: Documentation
---

## Widget Types

First of all we will have a look at what type of widgets can even be created, every one of them has its own drawbacks and advantages and are designed for specific use cases.
Widget type decide what kind of data we receive and what we can do with it.

The important thing to understand tough is that all widgets can execute any arbitrary `Rest API` call, over so called `Services`, which are described in the `Basic Widget API`, `Services` section.

![Widget Types](images/image.png)

### Time series

The first one is `Time Series`, which display historical values for a selected period of time and therefore always include the option to enable their own `time window`.

![History button](images/image-1.png)

![History](images/image-2.png)

The `data` that can be accessed is passed over `data sources`, which are [`Entities`](https://thingsboard.io/docs/user-guide/entities-and-relations/) and then additionally the `data keys` that should be accessible.

![Data source](images/image-3.png)

Thoose values and the assigned [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) can then be accessed, read and used to display or do other things in the widget itself.

```js
var index;
self.ctx.defaultSubscription.datasources[index];
self.ctx.defaultSubscription.data[index];
```

See [`Subscription object`](https://thingsboard.io/docs/user-guide/contribution/widgets-development/#subscription-object) for more information on the contained `json` data.

The assigned [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) is accessible over the given variable `datasources`, where `index` decides which of the assigned `data sources` is used. This is the case because multiple `data sources` can be assigned to some widgets.

The assigned `timeseries data keys` are accessible over the given variable `data`, which contain a `dataKey` dictionary with the `name` of the key and other settings for that specific key.

```json
dataKeys: [ //  array of keys (Array<DataKey>) (attributes or timeseries) of the entity used to fetch data 
   { // dataKey
        name: 'name', // the name of the particular entity attribute/timeseries 
        type: 'timeseries', // type of the dataKey. Can be "timeseries", "attribute" or "function" 
        label: 'Sin', // label of the dataKey. Used as display value (for ex. in the widget legend section) 
        color: '#ffffff', // color of the key. Can be used by widget to set color of the key data (for ex. lines in line chart or segments in the pie chart).  
        funcBody: "", // only applicable for datasource with type "function" and "function" key type. Defines body of the function to generate simulated data.
        settings: {} // dataKey specific settings with structure according to the defined Data key settings json schema. See "Settings schema section".
    },
    //...
]
```

It also contains the actual data with the `unix timestamp` and the `value` for each time the data was sent in the previously defined `time window`.

```json
data = [
    {
        datasource: {}, // datasource object of this data. See datasource structure above.
        dataKey: {}, // dataKey for which the data is held. See dataKey structure above.
        data: [ // array of data points
            [   // data point
                1498150092317, // unix timestamp of datapoint in milliseconds
                1, // value, can be either string, numeric or boolean  
            ],
            //...
        ]  
     },
    //...
]     
```

This specific feature of development is shared with `Latest Values` widgets, which will be talked about in more detail in the next section.

```js
self.typeParameters = function() {
    return {
        dataKeysOptional: true, // Whether this widget can be configured with datasources without data keys
        maxDataKeys: 0, // Maximum allowed datasources for this widget, -1 - unlimited
        maxDatasources: 1, // Maximum allowed data keys for this widget, -1 - unlimited
        datasourcesOptional: false // Wether this widget can be configured without datasources
    };
}
```

The `typeParameters` section, allows to configure the `data sources` and how many are allowed to be entered by a person configuring that widget.

If it makes no sense to accept more than one [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) as a `data source`, that can simply be set. It is the same for the amount of keys and if that value is required in the first place or not.

#### Action source

`Action source` means any interaction by a user on a widget, like pressing a button, row or even a header, these interactions can then be used to execute certain `actions` mentioned in the `Dashboard Widget Interaction`, `Action` section.

The `Action sources` supported by this widget type are:

- `Widget header button` (The new button with the given icon, on the top right of the widget is pressed)

### Latest values

Very similair to `Time Series` widget, but additionaly allows for `attributes`, where as `Time Series` widget are restricted to only using `telemetry` as possible enterable `data keys`.

This is the case, because as the name implies this widget is only meant to fetch the newest record of any given `data key`, meaning no history is fetched so it is not possible to display graphs from the past value history, because `attributes` do not save that historic data.

#### Action source

`Action source` means any interaction by a user on a widget, like pressing a button, row or even a header, these interactions can then be used to execute certain `actions` mentioned in the `Dashboard Widget Interaction`, `Action` section.

The `Action sources` supported by this widget type are:

- `Widget header button` (The new button with the given icon, on the top right of the widget is pressed)

Certain `Latest Values` widgets, also additionally contain more `action sources`, especially for pie charts.

- `On slice click` (One of the given slices of the pie chart is pressed)

### Control Widget (RPC)

Widgets that are specifically made to send [`Remote Procedure Call`](https://thingsboard.io/docs/user-guide/rpc/) to a given [`device`](https://thingsboard.io/docs/user-guide/ui/devices/) and handle the response if any appropriately.

This [`device`](https://thingsboard.io/docs/user-guide/ui/devices/) has to be set for the widget to work correclty, not unlike the `Latest Values` and `Time Series` widgets.

![Target device](images/image-4.png)


The access to the `RPC commands` is done over the `controlApi` component.

```js
// Name of the method that should be called on the target device
var methodName = "";
// Parameters that are passed to the method, can be null if the method needs to parameters
var optionalParameters = {};
// Timeout until the widget requests a response to the rpc request or it will fail
var optionalTimeout = 10000;
// Wether the rpc request is persistent or not, opposite of lightweight rpc,
// which only live for 30 seconds and since they are short-lived do not need to be saved into the database, consumes low amount of resources.
// Where as persistent rpc has a configurable lifetime and is stored in the database, meaning even if the device is not connected at the moment it will still handle the call once it is online again
// See https://thingsboard.io/docs/user-guide/rpc/#server-side-rpc for more information on persitent rpc
var optionalRequestPersistent = false;
// Polling interval of persistent rpc command response, meaning how many milliseconds we wait until we fetch if the persistent rpc has been responded too
var persistentPollingInterval = 5000;

var commandObservable = self.ctx.controlApi.sendOneWayCommand(methodName, optionalParameters, optionalTimeout);

commandObservable.subscribe(
    function (response) {
        var json = response.toString();
        document.getElementById('textArea').value = json;
        self.ctx.detectChanges();
    },
    function (rejection) {
        self.ctx.$scope.error = rejection.status + ": " + rejection.statusText;
        self.ctx.detectChanges();
    }
);

self.ctx.controlApi.sendTwoWayCommand(methodName, optionalParameters, optionalTimeout);

// Ensure to add this so if the widget is unloaded while the call is ongoing,
// it will clear the command and the aquired resources
self.onDestroy = function() {
    self.ctx.controlApi.completedCommand();
}
```

#### Action source

`Action source` means any interaction by a user on a widget, like pressing a button, row or even a header, these interactions can then be used to execute certain `actions` mentioned in the `Dashboard Widget Interaction`, `Action` section.

The `Action sources` supported by this widget type are:

- `Widget header button` (The new button with the given icon, on the top right of the widget is pressed)

### Alarm Widget

Like `Latest Values` and `Time Series`, this widget also uses a `data source`, but it is called differently, namely `alarm source`. Therefore it can also use a seperate `time window`.

But compared to the other widget this is not meant to display data directly, but instead display [Alarms](https://thingsboard.io/docs/user-guide/alarms/) generated by a given [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/).

[Alarms](https://thingsboard.io/docs/user-guide/alarms/) can be most easily configured on a [Device Profile](https://thingsboard.io/docs/user-guide/device-profiles/), if certain conditions are met. For example the temperature in a critical component rising to a very high temperature.

![Device Profile alarm rules](images/image-5.png)

#### Action source

`Action source` means any interaction by a user on a widget, like pressing a button, row or even a header, these interactions can then be used to execute certain `actions` mentioned in the `Dashboard Widget Interaction`, `Action` section.

The `Action sources` supported by this widget type are:

- `Action cell button` (The new button in all rows with [`Entities`](https://thingsboard.io/docs/user-guide/entities-and-relations/) is pressed)

![Action cell button](images/image-8.png)

- `Widget header button` (The new button with the given icon, on the top right of the widget is pressed)

### Static Widget

Unlike all other widget types, `static` widgets do not use any `data source` and simply offer the possiblity to create and opponent with custom `HTML` and `CSS` code.

![Action](images/image-6.png)

This does not mean this component can be underestimated tough, because it is the component with the most flexiblity when referring to `Action sources`.

#### Action source

`Action source` means any interaction by a user on a widget, like pressing a button, row or even a header, these interactions can then be used to execute certain `actions` mentioned in the `Dashboard Widget Interaction`, `Action` section.

The `Action sources` supported by this widget type are:

- `On HTML element click` (The `HTML` element with the given `name` or `id` is pressed)
- `Widget header button` (The new button with the given icon, on the top right of the widget is pressed)
