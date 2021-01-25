### BetterScroll 的bug

- bs的滚动框内如果有没加载完的图片或者其他文件，bs不会更新height，需要在加载图片后调用refresh方法

### $bus 自定义事件总线

- main.js中 vue实例定义一个$bus ,Vue.prototype.$bus = new Vue()

  使用：1）发送事件 this.$bus.$emit('eventName')

  ​			2) 接收并处理事件  this.$bus.on('eventName'，（) => {

  ​											  // do someting

  ​											})