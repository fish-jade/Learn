> 在瞎逛Vue官网时，发现一个组件选项属性model  
根据官网的说法是当自定义组件使用v-model时，可配置prop与event  
因为，在默认情况下,v-model所绑定的prop为value与事件input
```
// html
<box v-model="num"></box>{{num}}

等同于
<box :value="num" @input="val => {num=val}">


// script
Vue.component('box', {
  template: `<div><button @click="input()"></button>{{value}}</div>`,
  props: ['value'],
  methods: {
    input () {
      this.value++ 
      this.$emit('input', this.value) 
      // 因为vue已经写了定义input事件，只需要触发即可
    }
  }
})
new Vue({
  data () {
    return {
      num : 123
    }
  }
})

上诉的input事件，也是采取的eventBus的形式实现的
```

自定义组件 配置 model 属性
```
// html
<box v-model="num"></box>

等同于
<box :val="num" @add="val => {num=val+1}">


// script
Vue.component('box', {
  template: `<div><button @click="add()"></button>{{val}}</div>`,
  model:{
    prop: 'val',
    event: 'add'
  },
  props: ['val'],
  methods: {
    add () {
      this.$root.$emit('add', this.val) 
    }
  }
})
new Vue({
  created: {
    this.$root.$on('add',(val)=>{
      this.num = val + 1
    })
  },
  data () {
    return {
      num : 123
    }
  }
})
```
理解：  
  其实在自定义组件中使用v-model，只是让使用更加简洁了 。 因为v-model 是一个语法糖。  
  最终的实现父子组建的数据绑定也还是通过eventBus。