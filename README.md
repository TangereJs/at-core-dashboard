# at-core-dashboard

#### at-silver-* elements, actionListeners and at-core-dashboard.data-changed

Consider the following generic schema
```javascript
var schema = {
  properties: {
    text1: {
      type: 'string',
      title: "Text 1"
    },
    transform1: {
      type: 'element',
      xtype: "at-silver-transform",
      title: "Transform 1",
      template: "field1={{text1}}",
      actionListeners: [{
        value: "text1"
      }]
    },
    transform2: {
      type: 'element',
      xtype: "at-silver-transform",
      title: "Transform 2",
      template: "transform1={{transform1}}",
      actionListeners: [{
        value: "transform1"
      }]
    },
    transform3: {
      type: 'element',
      xtype: "at-silver-transform",
      title: "Transform 3",
      template: "transform2={{transform1}}",
      actionListeners: [{
        value: "transform2"
      }]
    },
    ...
    transform_n-1: {
      type: 'element',
      xtype: "at-silver-transform",
      title: "Transform n-1",
      template: "transform_n-2={{transform_n-2}}",
      actionListeners: ["transform_n-2"]
    },
    transformn: {
      type: 'element',
      xtype: "at-silver-transform",
      title: "Transform n",
      template: "transform_n-1={{transform_n-1}}",
      actionListeners: ["transform_n-1"]
    },
    dashboardEcho1: {
      type: 'element',
      xtype: "at-core-dashboard-echo",
      title: "Dashboard echo 1",
      template: "transform1={{transform1}}",
      actionListeners: [{
        value: "transform1"
      }]
    }
  }
}
```

When initial data is set with `at-core-dashboard.data = { text1="value1" }` the following needs to happen
  * set value of text1 to value1
  * update elements that have listeners

The schema presents the situation where we need to update elements that have listeners `n` times. We have a long waterfall to complete.
On top of this, replace any of the silver-transform elements with silver-datasource elements which return results asynchronously. Then waterfall must be completed asynchronously.
