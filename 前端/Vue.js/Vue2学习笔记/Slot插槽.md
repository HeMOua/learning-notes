# Slot插槽

## 匿名插槽

先定义一个匿名插槽组件SingleSlot.vue

```vue
<template>
	<div>
        <h3>匿名插槽组件</h3>
        <slot />
    </div>
</template>
```

使用

```vue
<div>
    <single-slot>
        匿名插槽
    </single-slot>
</div>
```

## 具名插槽

定义一个具名插槽组件NameSlot.vue

```vue
<template>
	<div>
        <h3>具名插槽组件</h3>
        <slot name="slot1" />
        <slot name="slot2" />
    </div>
</template>
```

使用

```vue
<div>
    <name-slot>
        <div slot="slot1">
            插槽1
        </div>
        <template v-slot:slot2>
            <span>插槽2</span>
        </template>
    </name-slot>
</div>
```

## 作用域插槽

新建文件ChildSlot.vue

```vue
<template>
	<div class="child-slot">
		<slot name="childSlot" :data="data" />
    </div>
</template>
<script>
export default {
    data() {
        return {
            data: [1, 2, 3, 4, 5]
        }
    }
}
</script>
```

使用

```vue
<div>
    <!-- 展示数据 -->
    <child-slot>
        <template slot="childSlot" slot-scope="scope">
			<ul>
                <li v-for="(item, index) in scope.data" :key="index">
                    {{ item }}
            	</li>
            </ul>
        </template>
    </child-slot>
    <!-- 直接显示数据 -->
    <child-slot v-slot:data="scope">
        {{ scope.data }}
    </child-slot>
    <!-- 不适用提供的数据，作用域倒退为匿名插槽 -->
    <child-slot>
        <template slot="childSlot" slot-scope="scope">
			m
        </template>
    </child-slot>
</div>
```

