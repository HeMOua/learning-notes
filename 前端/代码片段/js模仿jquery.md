js模仿jquery

```js
class jQuery {
  constructor(obj) {
    if (typeof obj === 'object') {
      this.elem = obj
    } else {
      this.elem = document.querySelectorAll(obj)
    }
  }

  css(propertyName, value) {
    if (!this.elem) return
    for (var i = 0, len = this.elem.length; i < len; i++) {
      this.elem[i].style[propertyName] = value
    }
  }

  cssText(value) {
    if (!this.elem) return
    for (var i = 0, len = this.elem.length; i < len; i++) {
      this.elem[i].style.cssText = value
    }
  }

  remove() {
    if (!this.elem) return
    for (var i = 0, len = this.elem.length; i < len; i++) {
      this.elem[i].remove()
    }
  }

  removeClass(obj) {
    if (!this.elem) return
    for (var i = 0, len = this.elem.length; i < len; i++) {
      this.elem[i].classList.add(obj)
    }
  }

  addClass(obj) {
    if (!this.elem) return
    for (var i = 0, len = this.elem.length; i < len; i++) {
      this.elem[i].classList.remove(obj)
    }
  }

  on(event, callback) {
    if (!this.elem) return
    for (var i = 0, len = this.elem.length; i < len; i++) {
      this.elem[i].addEventListener(event, function(e) {
        if (typeof callback === 'function') {
          callback(e)
        }
      })
    }
  }

  index(item) {
    for (let i = 0; i < this.elem.length; i++) {
      if (this.elem.item(i) === item) return i
    }
    return -1
  }
}

export default jQuery
```

