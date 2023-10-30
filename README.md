# @cybertale/interface

[![npm version](https://badge.fury.io/js/%40cybertale%2Finterface.svg)](https://badge.fury.io/js/%40cybertale%2Finterface)
[![Build Status](https://travis-ci.com/Joso997/cyber-interface.svg?branch=main)](https://travis-ci.com/Joso997/cyber-interface)
[![Coverage Status](https://coveralls.io/repos/github/Joso997/cyber-interface/badge.svg?branch=main)](https://coveralls.io/github/Joso997/cyber-interface?branch=main)
![License](https://img.shields.io/npm/l/%40cybertale%2Finterface.svg)

## Description

@cybertale/interface is a Web Development library that offers an ECS interface. 
Its purpose is to provide developers with a defined set of architectural guidelines, which, if followed, will turn their application into a data-driven one. Additionally, it can be utilized as an add-on with any JavaScript framework. 
For example, it can be incorporated with Vue.

**Keywords**
- Entity - carries all information to generate your component.
- Region - combined with mechanic it evolves into a powerful container which allows orderly manipulation of entities data.
- Mechanic - like in game development, a self-contained logic blocks for entity manipulation.

**Key Feature:**
- It ensures that each HTML element corresponds to a single code block.
- With SPA frameworks, you can separate your code by routes and make each route a self-contained region.
- For actual logic blocks, you can avoid duplicate code by following this principle consistently.
- One Form view/component to rule them all. (Brought to you by data-driven development)
- The API response will contain all the necessary data while keeping its size to a minimum.

## Installation

You can install @cybertale/interface using npm:
```
npm install @cybertale/interface
```
Alternatively, you can download the package from GitHub and include it in your project manually.

## Usage
It is necessary for you to define and link each component that you intend to utilize. 
The most recommended location for this task would be in your main.ts file.

Here's an example of how to use @cybertale/interface:

```typescript
import { ObjectType, ObjectTypeEnum } from '@cybertale/interface/src';

ObjectType.ObjectTypes[ObjectTypeEnum.Field](() => your_InputComponent)
ObjectType.ObjectTypes[ObjectTypeEnum.Button](() => your_ButtonComponent)
ObjectType.ObjectTypes[ObjectTypeEnum.Row](() => your_RowComponent)
(etc...)
```

After completing the task of defining and linking your components, the next step would be to create a folder named "mechanics". 
Within this folder, you would define your logic classes. 
These mechanic classes should be exclusively utilized for all Resource API requests, among other things.

Example:

```typescript
import { MechanicAbstract, ObjectTemplate, EventHandlerType } from '@cybertale/interface'

export namespace Manager.Mechanic{
    export class ExampleMechanic extends MechanicAbstract {
        public async InitGet (_id: string): Promise<ObjectTemplate[]> {
            // Write your API GET call here.
        }
        
        public InitSet (_objectTemplates: ObjectTemplate[]): ObjectTemplate[] {
          this.ObjectTemplates = _objectTemplates
          return this.ObjectTemplates
        }
        
        protected SubscribeConditions (): void {
          RegionType.RegionTypes[RegionEnum.TableColumn].ObjectTypes[ObjectTypeEnum.Button].SubscribeLogic(this.Button.bind(this))
        }
        
        public UnsubscribeConditions (): void {
          RegionType.RegionTypes[RegionEnum.TableColumn].ObjectTypes[ObjectTypeEnum.Button].NullifyLogic()
        }
        
        protected Button (eventHandler: EventHandlerType): void {
            switch (eventHandler.subObjectType) {
                case SubObjectTypeEnum.Left:
                    ...
                    break
                case SubObjectTypeEnum.Right:
                    ...
                    break
                default:
                    ...
                    break
            }
        }
    }
}
```
By using data-defined code, you can manage with just one file for View usage. However, for the sake of convenience, it is suggested to organize your views into separate files i.e. Form and Show.

Your folder structure could look like:

```
project
└───src
    └───views
    │   │   Form.vue
    │   │   Show.vue
    │   │   ...
    │
    └───mechanics
    │   │   tableMechanic.ts
    │   │   rowMechanic.ts
    │   │   formMechanic.ts
    │   │   ...
    │
    └───components
        └───formComponnets
        │    │  InputComponent.vue
        │    │  ButtonComponent.vue
        │    │  CheckBoxComponent.vue
        │    │  ...
        │             
        └───showComponents
             │  TableComponent.vue
             │  RowComponent.vue
             │  ColumnComponent.vue
             │  ...
```

Next add a component, as an example it is provided a hybrid component that is both an entity and a region (example is given as a Vue component):
```vue
<template>
  <tr v-if="renderComponent">
    <th scope="row"><img alt="arrow" width="27" src="../assets/arrow.png"></th>
    <component v-for="(_objectTemplate, key, index) in objectTemplates" :key="`${ key }-${ index }-${ Math.random().toString(36).slice(2, 7) }`"  :is="getComponent(_objectTemplate.Region, _objectTemplate.ObjectEnum)" :object='_objectTemplate'></component>
  </tr>
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component'
import { Manager } from '@/mechanics/rowMechanic'
import {
  ObjectTemplate,
  MechanicAbstract,
  ObjectType,
  StatTypeEnum,
  ObjectTypeEnum,
  RegionType,
  RegionEnum,
  SubObjectTypeEnum, ActionTypeEnum, StatType
} from '@cybertale/interface'
@Options({
  props: {
    entity: Array,
    index: Number,
    rerender: Function
  }
})
export default class RowComponent extends Vue {
  rerender!: () => void
  mechanic: MechanicAbstract = Manager.Mechanic.RowMechanic.getInstance(this.rerender.bind(this))
  regionEnum = RegionEnum
  statTypeEnum = StatTypeEnum
  objectTypeEnum = ObjectTypeEnum
  objectType = ObjectType
  renderComponent= false
  entity!: ObjectTemplate[]
  objectTemplates!: ObjectTemplate[]
  index!: number

  mounted () {
    this.objectTemplates = this.mechanic.InitSet(this.entity)
    if (this.objectTemplates !== undefined) {
      this.objectTemplates = this.mechanic.Append(
        [
          new ObjectTemplate(RegionEnum.TableColumn, ObjectTypeEnum.ColumnButton, SubObjectTypeEnum.ParentObject, ActionTypeEnum.None, {
            [StatTypeEnum.Id]: StatType.StatTypes[StatTypeEnum.Id]().CreateStat().InitData(this.objectTemplates[0].Stats[StatTypeEnum.Id].Data)
          })
        ]
      )
    }
    this.renderComponent = true
  }

  beforeUnmount () {
    this.mechanic.UnsubscribeConditions()
  }

  getComponent (_regionEnum : number, _objectEnum: number) {
    return RegionType.RegionTypes[_regionEnum].ObjectTypes[_objectEnum].GetComponent()
  }
}
</script>
```

NOTE: Currently, only one instance of a region type can be used multiple times per instance, provided that the region has entities subscribed to it.
```ts
mechanic: MechanicAbstract = Manager.Mechanic.RowMechanic.getInstance(this.rerender.bind(this))
```
If you intend to use a region only once per instance or if no entities are subscribed to it, you must use:
```ts
mechanic: MechanicAbstract = new Manager.Mechanic.FormMechanic(this.reRender.bind(this))
```
To illustrate the potential use of stats and how to link html events with entities, here is an example of a pure entity:
```vue
<template>
  <div class="mb-3 row justify-content-md-center">
    <div class="col-lg"></div>
    <div class="col input-group">
      <label :title="tooltipCase()" class="input-group-text" :hidden="specialCase()">{{object.Stats[statTypeEnum.Label].Data }}</label>
      <input class="form-control"
             :id="object.Stats[statTypeEnum.Tag].Data"
             :required="attributeCheck(statTypeEnum.Required)"
             :disabled="attributeCheck(statTypeEnum.Disabled)"
             :autocomplete="`${object.Stats[statTypeEnum.AutoComplete] !== undefined?object.Stats[statTypeEnum.AutoComplete].Data:''}`"
             :class="object.Stats[statTypeEnum.Design].Data+' '+validate()"
             :type="`${object.Stats[statTypeEnum.ElementType] !== undefined?object.Stats[statTypeEnum.ElementType].Data:''}`"
             :value="object.Stats[statTypeEnum.Value].Data"
             :placeholder="`${object.Stats[statTypeEnum.Placeholder].Data}`"
             @input="regionType.RegionTypes[object.Region].ObjectTypes[object.ObjectEnum].ChooseSubType(object, $event.target.value)">
      <slot></slot>
      <div class="invalid-feedback">{{ `${object.Stats[statTypeEnum.ErrorMessage] !== undefined?object.Stats[statTypeEnum.ErrorMessage].Data:''}` }}</div>
    </div>
    <div class="col-lg"></div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component'
import { ObjectTemplate, ObjectType, StatTypeEnum, ObjectTypeEnum, RegionType, RegionEnum } from '@cybertale/interface'
@Options({
  props: {
    object: ObjectTemplate
  }
})
export default class InputComponent extends Vue {
  statTypeEnum = StatTypeEnum
  objectTypeEnum = ObjectTypeEnum
  objectType = ObjectType
  regionType = RegionType
  regionEnum = RegionEnum
  object!: ObjectTemplate

  validate () : string {
    if (this.object.Stats[this.statTypeEnum.IsValid] === undefined) { return '' }
    if (this.object.Stats[this.statTypeEnum.IsValid].Data === '') { return '' }
    if (this.object.Stats[this.statTypeEnum.IsValid].Data) { return 'is-valid' }
    if (this.object.Stats[this.statTypeEnum.ErrorMessage].Data === null) { return '' }
    if (this.object.Stats[this.statTypeEnum.ErrorMessage].Data !== '') { return 'is-invalid' }
    return ''
  }

  specialCase () : boolean {
    if (this.object.Stats[this.statTypeEnum.ElementType] === undefined) { return false }
    return this.object.Stats[this.statTypeEnum.ElementType].Data === 'hidden'
  }

  attributeCheck (statType : number) : boolean | string {
    if (this.object.Stats[statType] === undefined) { return false }
    if (this.object.Stats[statType].Data === '') { return false }
    return this.object.Stats[statType].Data
  }

  tooltipCase () : string | undefined {
    if (this.object !== undefined) {
      if (this.object.Stats[this.statTypeEnum.Tooltip] !== undefined) {
        return this.object.Stats[this.statTypeEnum.Tooltip].Data
      }
    }
  }
}
</script>
```

## Access
All your Resource API responses will from now on be unified into a single json data block.
```json
[[
  {"Stats":
    [
      {"Data":"Name"},
      {"Data":"default"},
      {"Data":""},
      {"Data":"name"},{
      "Data":"bf0a1192-dad3-4305-aeec-a675d702020c"}
    ],"Region":1,"ObjectEnum":1,"SubObjectEnum":0,"ActionEnum":0}
]]
```
The above is an example of a single entity data. Based on this data program will generate an HTML representation as defined.
To access each individual data use:
```ts
this.object.Stats[statTypeEnum.Label].Data // "Name"
this.object.Stats[statTypeEnum.Value].Data // "default"
this.object.Stats[statTypeEnum.Design].Data // ""
this.object.Stats[statTypeEnum.Tag].Data // "name"
this.object.Stats[statTypeEnum.Id].Data // "bf0a1192-dad3-4305-aeec-a675d702020c"
```
To change visual and logical representation of the above data just change the entity enum types:
```ts
this.object.Region = RegionEnum.Form //Region acts as a container
this.object.ObjectEnum = ObjectTypeEnum.Button //Link to the component
this.object.SubObjectEnum = SubObjectTypeEnum.Middle //To diffirentiate between actions
this.object.ActionEnum = ActionTypeEnum.Click //Called manually or over linked html event
```
NOTE: depending on action type, it may even invoke a mechanic event if subscribed.

Example on how to subscribe to a HTML event (in Vue):
```vue
@input="regionType.RegionTypes[object.Region].ObjectTypes[object.ObjectEnum].ChooseSubType(object, $event.target.value)">
```

**List of all Object types:**
- Row,
- Field,
- Button,
- Text,
- Output,
- Alert,
- CheckBox,
- DataList,
- SelectList,
- Radio,
- Column,
- ColumnButton,
- FieldButton,
- SelectButton,
- ListRow,
- ECabinetRow,
- ECabinetColumn,
- ModalForm,
- MapPicker,
- FieldCode,
- DataSelect

**List of all Region types:**
- Form
- Table
- TableColumn
- TableRow
- Show
- Footer
- List
- ListRow
- ECabinet
- ECabinetRow
- ModalForm
- MapPicker

**List of all SubObject types:**
- ParentObject
- Middle
- Left
- Right
- Up
- Down

**List of all Stat types:**
- Label
- Value
- Design
- Tag
- Id
- ElementType
- Placeholder
- ItemList
- Tooltip
- Required
- Disabled
- AutoComplete
- BelongsTo
- ErrorMessage
- IsValid

**List of all Action types:**
- None
- Click
- Insert
- InsertUrl
- InsertClick
- InsertNumber
- Check
- SelectIdFromName

## Explanation
The data-driven nature of ECS calls for a departure from the traditional approach of nesting components. 
Instead, entities are treated as first-party components and organized into regions for logical separation. 
These regions can be associated with specific mechanics through event-based linkage, whereby entities within a region subscribe to a defined mechanic.

Visual Representation:
- Table Region - black
- Footer Region - black
- Row Region - blue
- Column Region - green
- Individual Entities - orange

![Alt text](./Untitled.png?raw=true "Title")

It's worth noting that certain components, similar to rows, can serve as both a region and an entity, making it simpler to nest HTML elements.
[For more information click me (translation TBA).](https://github.com/Joso997/implementacija-arhitektonskog-uzorka-entitetsko-komponentnog-sustava-za-razvoj-igara-i-web-aplikacij/blob/master/Dokumentacija%20Diplomskog%20Rada.pdf)
## Configuration
The @cybertale/interface package does not offer any configuration options.

## Known Issues
Depending on the IDE you use, to activate types you will be required to use @cybertale/interface/src in one of the imports.
You can use @cybertale/interface in all other imports.

Example:
```
import ... from '@cybertale/interface/src'
```
## Contributing
We value and welcome contributions from the community! 
To contribute to @cybertale/interface, please review our contributing guidelines.
TBA.

Current state of README is enough to give the package a try, but to develop a production ready application we advise users to check this [project.](https://github.com/Joso997/cybertale-interface)

## License
The @cybertale/interface package is released under the GPL-3.0-only License.
