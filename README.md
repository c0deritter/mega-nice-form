# Knight Form by Coderitter

A mega nice data structure describing your forms. It is easily extensible and you can use your domain objects to fill it conveniently.

## Install

`npm install knight-form`

## Overview

### Elements

There are exactly two elements to describe your form.

- `FormElement`: A general purpose form element that is not a field
- `Field`: A field representing a property on your object

We added two more classes which are basically empty sub classes from the upper two. They are there for better readability.

- `Form`: An empty sub class of `Field` but with an adjusted constructor
- `Button`: An empty sub class of `FormElement`

Here is a short example.

```typescript
var form = new Form('CharacterEditForm', 'Character').add(
  new Field('string', 'name'),
  new Field('number', 'level')
  new FormElement('buttons').add(
    new Button('submit'),
    new Button('reset')
  )
)
```

Because a form is basically a field you can combine forms into one another. Also you can have your form handle not just an object as its value but also an array or a primitive type.

### Form values

The fields in your form describe the structure of the object your form should be able to work with.

```typescript
var form = new Form('CharacterEditForm', 'Character').add(
  new Field('string', 'name'),
  new Field('number', 'level')
  new FormElement('buttons').add(
    new Button('submit'),
    new Button('reset')
  )
)

arne = {
  name: "Arne Steppat",
  level: 9001
}

form.value = arne
```

### Reset

The form comes with a reset feature.

```typescript
form.conserveOriginalValues()
form.reset()
```

### Find elements

You can convienently search for elements by their name.

```typescript
var form = new Form('CharacterEditForm', 'Character').add(
  new Field('string', 'name'),
  new Field('number', 'level')
  new FormElement('buttons').add(
    new Button('submit'),
    new Button('reset')
  )
)

form.find('name')
form.find('buttons.submit')
form.find('submit') // works
```

### Paths

The form element names are used to create neat little paths.

```typescript
var form = new Form('CharacterEditForm', 'Character').add(
  new Field('string', 'name'),
  new Field('number', 'level')
  new FormElement('buttons').add(
    new Button('submit'),
    new Button('reset')
  )
)

form.find('name').path === 'CharacterEditForm.name'
form.find('name').fieldPath === 'Character.name' // ignores FormElements

form.find('submit').path === 'CharacterEditForm.buttons.submit'
```

You can use them for translation to generate a nice translation key.

```
Character.name=Name
CharacterEditForm.buttons.submit=Submit
```

### Rendering

The special thing about our renderers is that we do not provide you some black box configuration magic. We give you the source code. This way you can look at it and understand for yourself. True to the motto when you can understand it you can extend it.

So basically you create a renderer for one project which is able to render your form exactly in that look of your application. So every time you render a form you will get the same result and if you want to change something about your forms you do it in exactly one place.

```html
<!-- React -->
<Form form={form} />

<!-- Angular -->
<form [form]="form" />
```

### Widget

You can use our Widget interfaces to get a minimal boiler plate for your own widgets.

```typescript
export interface Widget {
  title?: string
  invisible?: boolean
  disabled?: boolean
  required?: boolean
  [key: string]: any
}

export interface FieldWidget extends Widget {
  password?: boolean
}
```
### Customize form elements

All those elements are combined in a tree. To customize the elements of your form you can use one of the properties put in place to avoid subclassing.

- `valueType`: There is a set of standard value types but you may also use your own
- `more`: Append anything which _is not_ related to the look of the widget here
- `widget`: Append anything which _is_ related to the look of the widget here

### Adjust a form

There are some methods available to adjust a form.

- `keep`: Keeps a list of elements
- `drop`: Drops a list of elements

### Visitor

There is a visitor which can be used to visit every element of a form tree. It can be used to implement custom functionality.

```typescript
class FormTranslationVisitor extends FormVisitor<void> {
  visit(element: FormElement) {
    let path = element instanceof Field ? element.fieldPath : element.path
    element.widget.title = translate(path)
    this.visitDeeper(element)
  }
}
```

### JSON

We are using [knight-json](https://github.com/c0deritter/knight-json) to create an object which is ready to be transformed into a JSON string and back into not just naked objects but into the real corresponding classes of this package.

## Detailed
### FormElement

The `FormElement` is the base class for every element in the tree. Here are its properties.

- `parent`: Every element knows its parent. (You do not have to set it by yourself.)
- `elements`: Every element can have arbitrary many sub elements. (It is a tree.)
- `name`: Every element has a name which it can be referred to. Additionally to its core function you can use it for anything you need. This is the idea here. It is a data structure.
- `prototype`: It is not used in the base element. It is used in `Field` when having a list where you can add and remove items. Use it for whatever you think.
- `widget`: Here you can set view specific attributes. We provide a base class which you can extend.

This element is the starting point if you do not have to deal with use inputtable values. Create anything you like from it. Remember is is just a data structure. The renderer that you create will decide what to do with your custom elements.

Do not be afraid to extend it.

#### FormElement properties act as features

Setting values on the elements is always optional. Think of it like activating a feature if you set a certain property. If you do not need a certain feature the library will not mind.

For example if you leave out the name and you want to retreive that element by its name it will not find it. It will find nothing.

#### Paths

Every element has a path. It is composed of every element's name up to the root element.

```typescript
new Form('character').add( // character
  new FieldSet('general').add( // character.general
    new Field('string', 'name') // character.general.name
  )
)
```

If an element does not have a name it is ignored.

#### Retrieving elements

The `find()` method lets you retreive any element from any element. A dot notation is used to access fields down the chain.

```typescript
var form = form.add(
  new FieldSet('general').add(
    new Field('string', 'name'),
    new Field('number', 'level')
  )
)

form.find('general')
form.find('general.name')
form.find('general.level')
```

If an element which contains sub elements does not have a name it will still be taken into consideration.

```typescript
var form = form.add(
  new FieldSet('general').add(
    new Field('string', 'name'),
    new Row().add( // does not have a name
      new Button('clear')
    )
  )
)

form.find('general.clear') // ignores the row and still finds the element
```

At first the form tries to resolve the path gapless not considering container elements without a name. If this fails the form starts to search the whole tree matching the path allowing gaps. It will return the first element it found.

We use a very tolerant approach here in order to meet you. Even if you are being more loose in your specifications the form will still try to make the best out of it.

#### Parent

Because every element knows its parent there is some functionality available.

```typescript
element.parent
element.root // the root element which can be the form but has not to be
element.form // the next parent form up in the chain
element.path // the complete path of the element up the the root
```

#### Extending with additional properties

You can attach any property to any form element.

```typescript
var element = new FormElement
element.more.validators = [ new Required() ]
```

Use it to attach the data you need for further functionality like validation.

### Field

The field is your building block to describe the structure of the object your form should be able to cope with. Every field represents a property on your object. The field itself does not determine a specific widget. That makes this form library platform independent.

#### Field properties

The properties of the field.

- `type`: The type of the value
- `value`: The actual value
- `options`: An array of options to choose from

The pre-defined value types are defined in enum `ValueType`.

- `array`
- `boolean`
- `date`
- `number`
- `object`
- `string`

The type of property `type` is a string which is on purpose. Add as many new types as you like. Remember it all depends on your renderer and what it makes out of your given type.

Also do not be afraid to inherit from `Field`.

#### The field path

Fields additionally have a special path `fieldPath` which only takes fields into consideration. This is good for translation because that way the ids are stable. Imagine adding or removing a `FieldSet` and suddenly all translation ids are changing.

```typescript
new Form('character').add(
  new FieldSet('general').add(
    new Field('string', 'name') // character.name <- general is ignored
  )
)
```

#### Primitive fields

You determine the type of a field by setting its value type property. Here are the primitive ones.

- `boolean`
- `date`
- `number`
- `string`

The constructor for all of these look the same.

```typescript
var booleanField = new Field('boolean', 'name')
```

Just exchange the value type `boolean` with anything from the list above.

In the field the name additionally represents the name of an attribute on an object. It is key for mapping your form fields to actual object attributes.

#### Object fields

An object field has an object as its value. Add sub elements which will determine the properties of your sub object.

```typescript
new Form().add(
  new Field('object', 'skills').add(
    new Field('number', 'agility'),
    new Field('number', 'strength')
  )
)
```

Or add options for example if you just want to reference an object.

```typescript
var friendsField = new Field('object', 'friend')
friendsField.addOptions(
  new Option('Son Goku', { id: 1 }), // the value here is an object
  new Option('Son Gohan', { id: 2 })
)
```

This scenario of referencing ids could also be solved differently through a field of type `number`. It all depends on what you need.

#### Array fields

If you initialize a field as an array you will need to give it a prototypical element. Is is used to create elements for the existing array items and when adding new ones. The prototype can be any form element.

```typescript
new Field('array', 'favouriteFood', new Field('object').add(
  new Field('string', 'name'),
  new Field('string', 'place')
))
```

The prototypical element does not need to have a name otherwise every form element in the array field will have the same name. Also it can be any form element of course.

#### Setting values

Setting a value is just like this.

```typescript
var field = new Field('string')
field.value = 'a value'
```

You can also set unappropriate values.

```typescript
var field = new Field('string')
field.value = 33
```

Setting a whole object will result in all sub fields to be filled with values from this object if possible. The field which you want to set the object on needs to be of type `object` and the names of the sub fields need to match the property names of the object.

```typescript
var goku = {
  name: 'Son Goku',
  level: 9001
}

var character = new Field('object').add(
  new Row().add( // additional non field elements will be ignored.
    new Field('string', 'name'),
    new Field('number', 'level')
  )
)

character.value = goku

character.findField('name').value // === 'Son Goku'
character.findField('level').value // === 9001
```

If your value is an array and the field's type is `array` the field will remove any existing children and uses the `prototype` to create new children according to the contents of the given array value. If there is no prototype set the list of children will simply remain empty.

```typescript
var characters = [
  {
    name: 'Son Goku',
    level: 9001
  },
  {
    name: 'Vegeta',
    level: 17000
  }
]

var characterPrototype = new Field('object').add(
  new Field('string', 'name'),
  new Field('number', 'level')
)

var charactersField = new Field('array', characterPrototype)
charactersField.value = characters
```

#### Options

An option is basically a value and a label. 

```typescript
var option = new Option(value, 'label', disabled)
```

Set options on the field if you want to have the user to choose from a give set of possibilities. It can be rendered to a drop down for example but also a auto completion field is thinkable.

Activate the feature on a field by presenting an array of options.

```typescript
var options = [
  new Option(150, "Satan"),
  new Option(9001, "Son Goku"),
  new Option(100000000, "Omni-King", true) // it is disabled
]

var field = new Field("int64", "level", options)
```

### Form

The form is just a field. Thus it can have different values types.

```typescript
var objectForm = new Form() // a form representing an object (the value type does not have to be set explicitly)
var arrayForm = new Form('array') // a form representing an array
var numberForm = new Form('number') // a form representing a number

objectForm.value // an object
arrayForm.value // an array
numberForm.value // a number
```

Also they are easily combinable.

```typescript
objectForm.add(arrayForm, numberForm)
```

#### Visual properties

The form has two properties to describe a frame for the form.

```typescript
var form = new Form

form.title = "Character form"
form.addButtons(
  new Button("submit"),
  new Button("reset")
)
```

#### Form creation from different sources

You can create forms from a form definitions in JSON and in general from objects that have the same properties as the form elements including form elements itself.

```typescript
Form.load(jsonString) // from a JSON string
Form.load('path/to/file.json') // from a JSON inside a file
Form.load(new Form().add(...)) // from another form

Form.load({ // from a form alike object
  '@type': 'form',
  name: 'character'
})

```

In the latter case the `@type` property is important. Without it the correct form element class cannot be chosen.

### Buttons

work in progress...

#### Register for events

You can also register for button events.

```typescript
// let the form find the element and set a listener on it
form.listen('submit', button => {
  
})

// set listener directly on the element
form.find('submit').listen(button => {
  
})
```

### Visual elements

#### Row

A visual element to define that the contained elements should be in a row.

```typescript
var row = new Row('rowName').add(
  // any element
)
```

#### FieldSet

A visual element to define that the contained elements should be framed.

```typescript
var fieldSet = new FieldSet('fieldSetName').add(
  // any element
)
```

#### Create your own

Just extend `FormElement` and do whatever you need to do.

```typescript
class Separator extends FormElement {
}
```

### Behavioural elements

#### Mapping

The mapping element maps a key to a form element. The key can be really anything. Use it to your liking to create new behavioural elements tailored to your needs. It is just a data structure which can be used for any behaviour.

```typescript
var mapping = new Mapping().add(
  new KeyToElement('key', new Elements().add(
    /// ...
  ))
)
```

#### FieldValueMapping

This element maps the value of a field to a form element. If the field has the specified value the corresponding element will shown as if they were part of the parent element.

```typescript
var form = new Form().add(
  new Field('boolean', 'hasTail'),
  new FieldValueMapping('tailMapping', 'hasTail').add( // a mapping that reacts to the value of field 'hasTail'
    new KeyToElement(true, new Elements().add( // if the value of 'hasValue' is true show the following elements
      new Field('number', 'levelWithTail'),
      new Field('number', 'tailLength')
    )
  )
)
```

### Rendering

If you do not have any renderer code as a boilerplate for your new project you will want to write it on your own. Create an infrastructure consisting of widgets and their mapping to concrete components of your platform dependent UI framework.

#### Widgets

The widget that we ship has the following basic properties.

- `invisble`
- `disabled`
- `label`
- `required`
- `error`

Define any widget that you may need.

```typescript
class NumberInput extends Widget {
  public min: number
  public max: number
}
```

#### Templates

The way you render your form is different in every famework you use. Similar is the way we want you to work with renderers. In our universe renderers are not some black box components that you choose and which you can only extend in a mystical way. In our universe you download the source of one of our provided renderers and include it into your project. Most of the time this is one file. The renderer serves as a starting point from which on you extend it. Programmatically. Not through mystic configuration.

A renderer is really simple. It is just a mapping from a field to visual representation.

Here you can see an example for Angular. The following code snippet is the TypeScript part of an Angular component `form.component.ts`.

```typescript
@ViewChild('stringInput') private stringInputTemplate: TemplateRef<any>;
@ViewChild('numberInput') private numberInputTemplate: TemplateRef<any>;
@ViewChild('select') private selectTemplate: TemplateRef<any>;
@ViewChild('list') private listTemplate: TemplateRef<any>;
@ViewChild('object') private objectTemplate: TemplateRef<any>;
@ViewChild('form') private formTemplate: TemplateRef<any>;
@ViewChild('fieldSet') private fieldSetTemplate: TemplateRef<any>;
@ViewChild('checkbox') private checkboxTemplate: TemplateRef<any>;
@ViewChild('fieldValueMapping') private fieldValueMappingTemplate: TemplateRef<any>;
@ViewChild('dateInput') private dateInputTemplate: TemplateRef<any>;
@ViewChild('element') private elementTemplate: TemplateRef<any>;
```

This is the complete list of template references which are defined in the HTML part of the Angular component `form.component.html`. Here for example is the definition of an Angular template for the number input widget.

```html
<ng-template #checkbox let-field="element" let-widget="element.widget">
  <input matInput [(ngModel)]="field.value" [type]="number" [disabled]="widget.disabled" [placeholder]="widget.label" [min]="widget.min" [max]="widget.max" />
</ng-template>
```

If you want to change it just edit the file. You included it into your project for this purpose.

The next step is to determine the appropriate wiget for the field. Either the field has a widget object attached. In this case just return the corresponding template. Or determine the widget based on the data found on the field. Here you can see how the property `type` is used to determine the appropriate widget.

```typescript
public getWidget(element: FormElement): TemplateRef<any> {
  // in case the widget was specified explicitely
  if (element.widget != null) {
    return element.widget;
  }

  // in all other cases the widget is auto detected
  if (element instanceof Field) {
    if (element.type == 'number') {
      return new NumberInput(); // can contain meaningful defaults
    }
  }
}
```

The next step is a mapping from a widget to a template.

This is the way we do it. But you do not necessarily do it the same way. You can go nuts here. It all depends on your imagination. No deterministic configuration logic which prevents you from exrepssing what you really need. It is just good old programming. This is very accessible for anyone without the need to learn black box behaviour.

The next thing that you want to do is to react to a form submission. Exactly the way you need it to be. There is the `onSubmit()` method which is already implemented but if you do not like it then replace it. Or just extend it. It is up to you.

### Visitor

We have built a visitor which visits every element of the form tree.

```typescript
button(button: Button)
element(element: FormElement)
field(field: Field)
form(form: Form)
formFrame(formFrame: FormFrame)
fieldValueMapping(fieldValueMapping: FieldValueMapping)
```

Use it to implement your own functionality.

Run it like this.

```typescript
form.visit(new YourVisitor())
```

#### Translation

To use your favourite translation mechanism create a visitor.

```typescript
class TranslationVisitor extends FormVisitor {
  
  constructor(private translator: YourTranslator)

  element(element: FormElement) {
    translator.translate(element.path) // use the path to create a nice translation id
  }

  field(field: Field) {
    translator.translate(field.fieldPath) // use the field path for fields to be independent of tree changes
  }
}
```

And run it.

```typescript
form.visit(new TranslationVisitor())
```

#### Validation

Attach your validation objects or whatever it is directly on the form elements.

```typescript
var field = new Field('string', 'name')
field.validators = [
  new Required(), new MaxLength(255)
]
```

Then create a visitor and use your favourite validation framework.

```typescript
class ValidationVisitor extends FormVisitor {

  public valid: boolean

  field(field: Field) {
    var validators = field.validators
    validators.forEach(validator => this.valid &= validator.isValid(field.value))
  }
}
```

And then validate.

```typescript
var validationVisitor = new ValidationVisitor();
form.visit(validationVisitor)

if (! validationVisitor.valid) {
  // ...
}
```
