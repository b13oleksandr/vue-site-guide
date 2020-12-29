#Vue site guide





# Install
### Yarn
``` sh
yarn add vue-site-guide
```
### NPM
``` sh
npm i vue-site-guide
```





# Props

## config
Guide configuration

**Type:** `Object`

Accepted values:

| Property name          | Type            | Default       | Description                                                             |
| :----------------------|:----------------|:--------------|:------------------------------------------------------------------------|
| steps                  | `Array `        | `[]`          | Configuration of guide steps `{ selector: '', title: '', message: '' }` |
| initialStep            | `Number`        | `0`           | Index number of initial step                                            |
| backdropClickToClose   | `Boolean`       | `true`        | Close guide by clicking outside of tooltip                              |
| prevButtonText         | `String`        | `Prev`        | The text that will be shown on the button Previous                      |
| prevButtonClass        | `String`        | `''`          | CSS classes for the button Previous                                     |
| prevButtonStyle        | `Object`        | `{}`          | Object with CSS properties and values for styling the button Previous   |
| nextButtonText         | `String`        | `Next`        | The text that will be shown on the button Next                          |
| nextButtonClass        | `String`        | `''`          | CSS classes for the button Next                                         |
| nextButtonStyle        | `Object`        | `{}`          | Object with CSS properties and values for styling the button Next       |
| skipButtonText         | `String`        | `Skip`        | The text that will be shown on the button Skip                          |
| skipButtonClass        | `String`        | `''`          | CSS class name of the button Skip                                       |
| skipButtonStyle        | `Object`        | `{}`          | Object with CSS properties and values for styling the button Skip       |
| tooltipAlign           | `String`        | `center`      | Tooltip alignment relative to step                                      |
| titleClass             | `String`        | `''`          | CSS classes for the tooltip title                                       |
| titleStyle             | `String`        | `''`          | Object with CSS properties and values for styling the tooltip title     |
| messageClass           | `String`        | `''`          | CSS classes for the tooltip message                                     |
| messageStyle           | `String`        | `''`          | Object with CSS properties and values for styling the tooltip message   |
| tooltipBgColor         | `String`        | `''`          | Background color of tooltip                                             |
| tooltipRadius          | `String`        | `''`          | Border radius of tooltip in CSS units                                   |
| tooltipPadding         | `String`        | `''`          | Padding of tooltip in CSS units                                         |
| tooltipShadow          | `String`        | `''`          | Shadow of tooltip in CSS units                                          |
| supportBgColor         | `String`        | `''`          | Background color of the step backdrop                                   |
| supportRadius          | `String`        | `''`          | Border radius of the step backdrop in CSS units                         |
| supportShadow          | `String`        | `''`          | Shadow of the step backdrop                                             |





# Events

## shown
Emitted after component has been shown to user

## stepChanged
Emitted when currently step was changed

**Payload**

Current step index (after step was changed) `Number`

## nextStep
Emitted when the user clicks Previous button

**Payload**

Current step index `Number`

## prevStep

**Payload**

Current step index `Number`

## beforeClose

Emitted immediately after the start of the closure of the guide

## close

Emitted after the guide was closed

## isBeginning

Emitted after the guide shown with initial-step is equal 0

## isEnd

Emitted after the guide shown with initial-step is equal of index of the last step

## reachBeginning

Emitted when guide reaches its initial step

## reachEnd

Emitted when guide reaches its last step





# Slots

## default

Default slot used for pass the content

```vue
<vue-site-guide v-model="guide" :config="guideConfig">
  <div id="title">
    ...
  </div>
    
  <div id="start">
    ...
  </div>
    
  <div id="features">
    ...
  </div>
</vue-site-guide>
```





# Scoped slots

## buttonPrev

Slot for Previous button

**Scope**

| Name                   | Type            | Description                                                             |
| :----------------------|:----------------|:------------------------------------------------------------------------|
| prev                   | `Function`      | Go to previous step                                                     |
| canPrev                | `Boolean`       | Set `false` if step index number is 0                                   |

**Example**

```vue
<vue-site-guide :config="config" v-model="guide">
  <template #buttonPrev="{ prev, canPrev }">
    <button
      @click="prev"
      :disabled="!canPrev"
    >
      Back
    </button>
  </template>

  ...
</vue-site-guide>
```


## buttonNext

Slot for Next button

**Scope**

| Name                   | Type            | Description                                                             |
| :----------------------|:----------------|:------------------------------------------------------------------------|
| next                   | `Function`      | Go to next step                                                         |
| canNext                | `Boolean`       | Set `false` if step index number is equal of index number of the last step                                                                        |

**Example**

```vue
<vue-site-guide :config="config" v-model="guide">
  <template #buttonNext="{ next, canNext }">
    <button
      @click="next"
      :disabled="!canNext"
    >
      Forward
    </button>
  </template>

  ...
</vue-site-guide>
```


## buttonSkip

Slot for Next button

**Scope**

| Name                   | Type            | Description                                                             |
| :----------------------|:----------------|:------------------------------------------------------------------------|
| skip                   | `Function`      | Close guide                                                             |


## {stepIndex}Title

Slot for tooltip title

**Example**

```vue
<vue-site-guide :config="config" v-model="guide">
  <template #0Title>
    <h2 style="color: #fff">
      Doc title
    </h2>
  </template>

  <template #1Title>
    <h2 style="color: #fff">
      Features
    </h2>
  </template>

  ...
</vue-site-guide>
```

## {stepIndex}Message

Slot for tooltip message

**Example**

```vue
<vue-site-guide :config="config" v-model="guide">
  <template #2Message>
    <div style="margin-bottom: 8px;">
      About your features
    </div>
    <img 
      style="max-width: 100%; border-radius: 3px; margin-bottom: 12px;" 
      src="https://picsum.photos/seed/picsum/400/200"
      alt=""
    >
  </template>

  ...
</vue-site-guide>
```
