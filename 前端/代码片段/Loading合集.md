# Loading合集

## 1

绝对好看

```vue
<template>
  <div v-if="loading" class="loading">
    <div class="loader" />
  </div>
</template>

<script>
export default {
  props: {
    loading: {
      type: Boolean,
      required: true
    }
  }
}
</script>
<style scoped lang="scss">
.loading {
    position: relative;
    width: 100%;
    height: 100%;
    max-height: 700px;
    min-height: 150px;
}
.loader {
    position: absolute;
    top: calc(50% - 1.25em);
    left: calc(50% - 1.25em);
    width: 2.5em;
    height: 2.5em;
    transform: rotate(165deg);

    &::before, &::after {
        content: '';
        position: absolute;
        top: 50%;
        left: 50%;
        display: block;
        width: 0.5em;
        height: 0.5em;
        border-radius: 0.25em;
        transform: translate(-50%, -50%);
    }

    &::before {
        animation: before-ani 2s infinite;
    }

    &::after {
        animation: after-ani 2s infinite;
    }
}

@keyframes before-ani {
    0% {
        width: 0.5em;
        box-shadow: 1em -0.5em rgba(225, 20, 98, 0.75), -1em 0.5em rgba(111, 202, 220, 0.75);
    }
    35% {
        width: 2.5em;
        box-shadow: 0 -0.5em rgba(225, 20, 98, 0.75), 0 0.5em rgba(111, 202, 220, 0.75);
    }
    70% {
        width: 0.5em;
        box-shadow: -1em -0.5em rgba(225, 20, 98, 0.75), 1em 0.5em rgba(111, 202, 220, 0.75);
    }
    100% {
        box-shadow: 1em -0.5em rgba(225, 20, 98, 0.75), -1em 0.5em rgba(111, 202, 220, 0.75);
    }
}
@keyframes after-ani {
    0% {
        height: 0.5em;
        box-shadow: 0.5em 1em rgba(61, 184, 143, 0.75), -0.5em -1em rgba(233, 169, 32, 0.75);
    }
    35% {
        height: 2.5em;
        box-shadow: 0.5em 0 rgba(61, 184, 143, 0.75), -0.5em 0 rgba(233, 169, 32, 0.75);
    }
    70% {
        height: 0.5em;
        box-shadow: 0.5em -1em rgba(61, 184, 143, 0.75), -0.5em 1em rgba(233, 169, 32, 0.75);
    }
    100% {
        box-shadow: 0.5em 1em rgba(61, 184, 143, 0.75), -0.5em -1em rgba(233, 169, 32, 0.75);
    }
}
</style>
```

## 2

绿色转圈，也挺好看

```html
<div class="loadEffect">
    <span></span>
    <span></span>
    <span></span>
    <span></span>
    <span></span>
    <span></span>
    <span></span>
    <span></span>
</div>

<style>
    .loadEffect{
        width: 100px;
        height: 100px;
        position: relative;
        margin: 0 auto;
        margin-top:100px;
    }
    .loadEffect span{
        display: inline-block;
        width: 20px;
        height: 20px;
        border-radius: 50%;
        background: lightgreen;
        position: absolute;
        -webkit-animation: load 1.04s ease infinite;
    }
    @-webkit-keyframes load{
        0%{
            -webkit-transform: scale(1.2);
            opacity: 1;
        }
        100%{
            -webkit-transform: scale(.3);
            opacity: 0.5;
        }
    }
    .loadEffect span:nth-child(1){
        left: 0;
        top: 50%;
        margin-top:-10px;
        -webkit-animation-delay:0.13s;
    }
    .loadEffect span:nth-child(2){
        left: 14px;
        top: 14px;
        -webkit-animation-delay:0.26s;
    }
    .loadEffect span:nth-child(3){
        left: 50%;
        top: 0;
        margin-left: -10px;
        -webkit-animation-delay:0.39s;
    }
    .loadEffect span:nth-child(4){
        top: 14px;
        right:14px;
        -webkit-animation-delay:0.52s;
    }
    .loadEffect span:nth-child(5){
        right: 0;
        top: 50%;
        margin-top:-10px;
        -webkit-animation-delay:0.65s;
    }
    .loadEffect span:nth-child(6){
        right: 14px;
        bottom:14px;
        -webkit-animation-delay:0.78s;
    }
    .loadEffect span:nth-child(7){
        bottom: 0;
        left: 50%;
        margin-left: -10px;
        -webkit-animation-delay:0.91s;
    }
    .loadEffect span:nth-child(8){
        bottom: 14px;
        left: 14px;
        -webkit-animation-delay:1.04s;
    }
</style>
```

