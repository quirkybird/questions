## 事件发布订阅

```JavaScript
class EventBus {
    on(eventName, listener) {}
    off(eventName, listener) {}
    once(eventName, listener) {}
    trigger(eventName) {}
}

const e = new EventBus();
// fn1 fn2
e.on('e1', fn1)
e.once('e1', fn2)
e.trigger('e1') // fn1() fn2()
e.trigger('e1') // fn1()
e.off('e1', fn1)
e.trigger('e1') // null
```

## 实现如下

````JavaScript
 //声明类
      class EventBus {
        constructor() {
          this.eventList = {} //创建对象收集事件
        }
        //发布事件
        $on(eventName, fn) {
          //判断是否发布过事件名称? 添加发布 : 创建并添加发布
          this.eventList[eventName]
            ? this.eventList[eventName].push(fn)
            : (this.eventList[eventName] = [fn])
        }
        //订阅事件
        $emit(eventName) {
          if (!eventName) throw new Error('请传入事件名')
          //获取订阅传参
          const data = [...arguments].slice(1)
          if (this.eventList[eventName]) {
            this.eventList[eventName].forEach((i) => {
              try {
                i(...data) //轮询事件
              } catch (e) {
                console.error(e + 'eventName:' + eventName) //收集执行时的报错
              }
            })
          }
        }
        //执行一次
        $once(eventName, fn) {
          const _this = this
          function onceHandle() {
            fn.apply(null, arguments)
            _this.$off(eventName, onceHandle) //执行成功后取消监听
          }
          this.$on(eventName, onceHandle)
        }
        //取消订阅
        $off(eventName, fn) {
          //不传入参数时取消全部订阅
          if (!arguments.length) {
            return (this.eventList = {})
          }
          //eventName传入的是数组时,取消多个订阅
          if (Array.isArray(eventName)) {
            return eventName.forEach((event) => {
              this.$off(event, fn)
            })
          }
          //不传入fn时取消事件名下的所有队列
          if (arguments.length === 1 || !fn) {
            this.eventList[eventName] = []
          }
          //取消事件名下的fn
          this.eventList[eventName] = this.eventList[eventName].filter(
            (f) => f !== fn
          )
        }
      }
      const event = new EventBus()

      let b = function (v1, v2, v3) {
        console.log('b', v1, v2, v3)
      }
      let a = function () {
        console.log('a')
      }
      event.$once('test', a)
      event.$on('test', b)
      event.$emit('test', 1, 2, 3, 45, 123)

      event.$off(['test'], b)

      event.$emit('test', 1, 2, 3, 45, 123)
      ```
````
