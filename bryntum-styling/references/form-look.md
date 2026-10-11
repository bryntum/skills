# A form that looks like a form

The defaults don't give a panel of fields a form look. Three things do:

```javascript
new Panel({
    title         : 'Employee',
    width         : 480,                  // give a form its own width (400–560); a flex layout stretches it
    labelPosition : 'align-before',       // labels in one column, inputs in another; 'before' rags the inputs
    items         : [
        { type : 'textfield', label : 'Full name',  name : 'name' },
        { type : 'combo',     label : 'Department', name : 'dept', items : ['Sales', 'Ops'] },
        { type : 'datefield', label : 'Hire date',  name : 'hired' }
    ],
    bbar : [
        '->',
        { type : 'button', text : 'Reset' },
        { type : 'button', text : 'Submit', rendition : 'tonal' }  // primary action only
    ]
});
```

`labelPosition` is a Container config (`'before'`, `'above'`, `'align-before'`). Set on the panel it applies to every field inside, including ones added later.

For a dialog, use `Popup` instead of `Panel` with the same `items` and `labelPosition`: `modal : true, centered : true, closeAction : 'hide'`. Read `isValid` before saving. To reset the fields between opens, assign `popup.values = { … }`: `resetValues()` throws `Cannot set properties of undefined (setting 'optionsProp')` on a Popup in 7.3.7. An empty `required` field is flagged (error tip, invalid state) as soon as the popup shows. To flag it only on Save, omit `required` and call `field.setError('…', false, true)` in the save handler. `values` is typed `Record<string, object>`, so plain data needs a cast under strict TypeScript.
