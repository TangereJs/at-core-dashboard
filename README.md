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

#### Details
When at-core-dashboard.dataChanged is executed newValue holds the values that should be set on elements. To achieve this
  * `_isBatchUpdate` flag is set to true
  * then newValue properties are enumerated and iterated over
  * in each iteration newValue's property name is the id of the element on the form and newValue[property name] holds the value that should be set
  * setting the new value on the element (at-form-text for example) triggers value-changed event for that element; to prevent infinite loops and stackoverflow errors, while `_isBatchUpdate` flag is true, all value-changed events are suppressed
  * this works just fine in at-core-form and at-form-complex, but this suppression is a big problem for actionListeners feature in at-core-dashboard
  * after each element's new value from newValue is applied to elements, elements that have action listeners are updated; while this is happening `_isBatchUpdate` flag is still true; if its false that produces infinite loop and stack overflow error.
  * now, at-silver-transform will fire value-changed at the end of its actionListener function; this event will be suppressed because `_isBatchUpdate` flag is true
  * since value-changed events are suppressed this produces incorrect update of element that have action listeners since at-core-dashboard.data is never up to date
  * an alternate approach is required to solve this problem
  * on top of this, the whole scenario holds true for at-silver-datasource which fires value-changed event asynchronously
  * So, not only is at-core-dashboard.data never up to date, it is never up to date asynchronously as well.
  * Thus, asynchronous situation needs to be handled
