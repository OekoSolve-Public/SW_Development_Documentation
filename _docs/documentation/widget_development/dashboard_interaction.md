---
layout: default
title: Dashboard Widget Interaction
parent: ThingsBoard Widget Development
grand_parent: Documentation
nav_order: 4
---

## Dashboard Widget Interaction

When a widget is added to a [`Dashboard`](https://thingsboard.io/docs/user-guide/dashboards/), it allows for additional interactions.

### Copy Value / Reference

When right-clicking in edit mode on a widget it is possible to either enter edit mode, copy the widget, copy the reference to the widget or delete it.

![Widget context](https://raw.githubusercontent.com/OekoSolve-Public/SW_Development_Documentation/main/_images/widget_context.png)

The most important thing to understand is the difference between `Copy` and `Copy Reference`.

Not unsimilair to the `Copy value` vs. `Copy Reference` in code, the simple `Copy` button will just copy the value and when pasted into another [`Dashboard`](https://thingsboard.io/docs/user-guide/dashboards/) or the same one paste the values, there is no conenction between the two widgets.

Meaning if one is changed or deleted the other one stays the same. However when using the `Copy reference` button, it is possible to create a link between the copied and pasted widget.

Meaning if one is changed or deleted the other one is changed or deleted as well. This however can only be done in the same [`Dashboard`](https://thingsboard.io/docs/user-guide/dashboards/), but allows to share widget settings and values between multiple [`Dashboard States`](https://thingsboard.io/docs/user-guide/dashboards/#states) in the same [`Dashboard`](https://thingsboard.io/docs/user-guide/dashboards/).

### Data

The data tab contains the possiblity to add a `data source` and when using `ThingsBoard Professional Edition` under the header `Data Settings` the possiblity to enable `data export` of the given widget into `.csv`, `.xls` or `.xslx` format.

#### Data key configuration

When a `data source` was added as well as a `data key` it is possible to press the edit button and change even more configuration about it like an additional symbol that should be displayed next to the value or the amount of digits after the floating point.

Especially interesting is the possiblity, when data keys have been added and `ThingsBoard Professional Edition` is used, to use the `data post processing function`. Only possible for `shared` or `client` attributes and `telemetry` values.

This allows to execute arbitrary `JavaScript` code before the value is displayed on the widget.

![Data key configuration](https://raw.githubusercontent.com/OekoSolve-Public/SW_Development_Documentation/main/_images/data_key_config.png)

Additionally for the `data post-processing`, these parameters are passed. 

```js
time: number - timestamp in milliseconds of the current datapoint.
value: primitive (number/string/boolean) - A value of the current datapoint.
prevValue: primitive (number/string/boolean) - A value of the previous datapoint after applied post-processing.
timePrev: number - timestamp in milliseconds of the previous datapoint value.
prevOrigValue: primitive (number/string/boolean) - An original value of the previous datapoint.
```

An example would be getting the relative difference between the current and previous uploaded data point.

```js
if (prevOrigValue) {
    return (value - prevOrigValue) / prevOrigValue;
} else {
    return 0;
}
```

-------------------

Furthermore the `advanced` tab under the `data key` tab allows to change even more settings about the `data key`. Configuration mentioned below only exists with the `Entities table` widget, but other widgets might contain more configuration about the `data key` as well.

This section contains, both the `cell style function`, which allows to change how the [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) cell should be styled and these parameters are passed.

```js
value: any - An entity field value displayed in the cell.
entity: EntityData - An EntityData object presenting basic entity properties (ex. id, entityName) and
provides access to other entity attributes/timeseries declared in widget datasource configuration.
ctx: WidgetContext - A reference to WidgetContext that has all necessary API and data used by widget instance.
```

And the `cell content function`, which allows to change how the [`Entity`](https://thingsboard.io/docs/user-guide/entities-and-relations/) value should be displayed and these paremters are passed. This function can not use the `Services`, because it directly expects a `return` with the result when the method is called and not once the result of the `Rest API` has been received.

```js
value: any - An entity field value displayed in the cell.
entity: EntityData - An EntityData object presenting basic entity properties (ex. id, entityName) and
provides access to other entity attributes/timeseries declared in widget datasource configuration.
ctx: WidgetContext - A reference to WidgetContext that has all necessary API and data used by widget instance.
```

An example would be displaying a colored red or green circle depending on a boolean value.

```js
var color;
var active = value;
if (active == 'true') {
  color = '#27AE60';
} else {
  color = '#EB5757';
}
return '<span style="font-size: 18px; color: ' + color + '">&#11044;</span>';
```

### Settings

The `settings` tab contains general configurations for all widgets, including a shadow the tile and an optional icon.

This setting can be ignored most of the time, besides the title which per default shows the name of the widget in the format `New <Widget Title>` and needs to be changed most of the time.

### Advanced

The `advanced` tab can be completly barren or the most important section of the widget, this is the case because this section is configured by the widget creator itself, mentioned in more detail [here](https://oekosolve-public.github.io/SW_Development_Documentation/docs/documentation/widget_development/basic_widget_api/#widget-editor-overview).

### Actions

The `actions` tab contains the possible type of `action` that can happen when interacting with an `action source`. The possible `action sources` are mentioned under the specific `widget types` because depending on the widget they vary greatly. Nevertheless all of these share the same possible types of `actions` to do if the interaction was called those being.

- `Navigate to new dashboard state` (Opens the given [`dashboard state`](https://thingsboard.io/docs/user-guide/dashboards/#states) as a subdashboard, meaning it is displayed like below, and allows jumping back to the dashboard that we were originally in)

![Subdashboard](https://raw.githubusercontent.com/OekoSolve-Public/SW_Development_Documentation/main/_images/sub_dashboard.png)

- `Update current dashboard state` (Opens the given [`dashboard state`](https://thingsboard.io/docs/user-guide/dashboards/#states) directly, meaning it will not be able to jump back like with the `Navigate to new dashboard state` option)

- `Navigate to other dashboard` (Opens a completly seperate configurable dashboard in a given [`dashboard state`](https://thingsboard.io/docs/user-guide/dashboards/#states) in a new browser tab or the same one)

- `Custom action` (Allows to execute arbitray `JavaScript` code, when the button is pressed)

Additionaly for the `custom action`, these parameters are passed. For static widgets the `entityId`, `entityName`, `additionalParams` and `entityLabel`, because those will be `undefined`, because they are only aquired if the widget has a `data source`.

```js
$event: MouseEvent - Result of a mouse click event
widgetContext: WidgetContext - WidgetContext that has all necessary API and data used by widget instance.
entityId: string - An optional string id of the target entity.
entityName: string - An optional string name of the target entity.
additionalParams: {[key: string]: any} - An optional key/value object holding additional entity parameters. 
entityLabel: string - An optional string label of the target entity.
```

- `Custom action (with HTML template)` (Very similair to `custom action`, but additional allows to display custom `HTML` and `CSS`)

- `Mobile action` (Allows to execute multiple different actions on a mobile phone, when using the `ThingsBoard` app)
