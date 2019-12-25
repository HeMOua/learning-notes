# Sync的使用

## 案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
</head>
<body>
	<div id="app">
		<h2>父组件</h2>
		<div v-show="show">我在这里</div>
		<hr>
		<child-item :show="show"></child-item>
	</div>

	<template id="temp">
		<div>
			<h2>子组件</h2>
			<button @click="handleClick">点我</button>
		</div>
	</template>

	<script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.14/vue.min.js"></script>
	<script>
		Vue.component('child-item', {
			props: ['show'],
			template: '#temp',
			methods: {
				handleClick() {
					this.show = !this.show
				}
			}
		})
		new Vue({
			el: '#app',
			data() {
				return {
					show: true
				}
			}
		})
	</script>
</body>
</html>
```

简单介绍一下，父组件里有个属性`show`

子组件在使用的时候将父组件的`show`属性给传递过去了，并且子组件有个方法想要修改父组件`show`的属性值

## 错误做法

注意看子组件的`handleClick`方法

```js
handleClick() {
    this.show = !this.show
}
```

因为父组件传递给子组件的值是单向的，所以当点击按钮后并没有任何效果

## 正确做法

1. 因为`show`属性属于父组件，想要修改它需要通过父组件，于是在父组件中定义一个修改`show`属性的`update`方法
2. 将定义的`update`方法传递到子组件上去

修改使用子组件的地方

```vue
<child-item :show="show" @update="update"></child-item>
```

在父组件中添加方法以及修改子组件方法

```html
<script>
    Vue.component('child-item', {
        props: ['show'],
        template: '#temp',
        methods: {
            handleClick() {
                this.$emit('update', !this.show)
            }
        }
    })
    new Vue({
        el: '#app',
        data() {
            return {
                show: true
            }
        },
        methods: {
            update(val) {
                this.show = val
            }
        }
    })
</script>
```

这样会发现一个缺点就是，子组件想要修改父组件的一个属性，还要再父组件中定义一个很弱智，但是不得不需要的方法，比较麻烦，所以做出了以下改进

## 小改进

在使用子组件的地方

```vue
<child-item :show="show" @update="show = $event"></child-item>
```

而在父组件中的update方法就可以删除了，修改完的父组件如下

```vue
<script>
    new Vue({
        el: '#app',
        data() {
            return {
                show: true
            }
        }
    })
</script>
```

可以看到省略掉了多余的update方法

## sync雏形

稍微修改一个传递给子组件的方法名

```vue
<child-item :show="show" @update:show="show = $event"></child-item>
<script>
    Vue.component('child-item', {
        props: ['show'],
        template: '#temp',
        methods: {
            handleClick() {
                this.$emit('update:show', !this.show)
            }
        }
    })
</script>
```

## sync的使用

通过观察sync的雏形，能发现`@update:show="show = $event"`，这句代码的功能十分单一、固定，因此vue自己就设置一个修饰符`sync`，替代了它的功能，修改后代码如下

```vue
<child-item :show.sync="show"></child-item>
<script>
    Vue.component('child-item', {
        props: ['show'],
        template: '#temp',
        methods: {
            handleClick() {
                this.$emit('update:show', !this.show)
            }
        }
    })
</script>
```

当然需要注意的是在组件中仍然需要手动执行`this.$emit(update:[属性值], [val])`这句代码

## 注意

在传递自定义方法时，如下

```vue
<child-item @update="update"></child-item>
```

其实相当于

```vue
<child-item @update="update($event)"></child-item>
```

