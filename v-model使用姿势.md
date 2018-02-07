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
      // 因为vue已经写了定义input事件并且所$on的input事件存在于当前实例上，只需要触发即可
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

上述的input事件，也是采取的eventBus的形式实现的
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
  created() {
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
  但是vue本来实现了，input change等事件，可以不用在写this.$root.$on 对 input change 事件进行监听。  
  可以直接触发input change 时间 对 父子组件中的数据进行双向绑定。

## 错误理解修正

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <script src="https://cdn.jsdelivr.net/npm/vue "></script>
  <style>
    #app div {
      margin: 20px 0;
    }
  </style>
  <title>Document</title>
</head>
<body>
  <div id="app">
    <div>
      <box v-model="type" :val = "type"></box>
    {{type}}
    </div>
    <div>
      <btn v-model="btn"></btn>
    {{btn}}
    </div>
    <div>
      <boxx v-model="boxx"></boxx>{{boxx}}
    </div>
    
  </div>
  <script>
    Vue.component('boxx', {
      template: `<div><button @click="add()">+</button>{{val}}</div>`,
      model:{
        prop: 'val',
        event: 'change'
      },
      props: ['val'],
      methods: {
        add () {
          console.log(this.val)
          this.val ++
          // this.$root.$emit('add', this.val) 
        }
      },
      watch: {
        val: function (val, oldVal) {
          console.log('this',this)
          console.log('this.root',this.$root)
          console.log('this.emit',this.$emit)
          this.$emit('change', this.val) 
        }
      }
    })
    Vue.component('btn', {
      template: `<button @click="input()">+</button>`,
      model: {
        prop: 'val',
        event: 'change'
      },
      props: ['val'],
      methods: {
        input(){
          this.val+='a'
          console.log(this.val)
          this.$emit('change',this.val)
        }
      }
    })
    Vue.component('box', {
      template: `<div>val:<input v-model="val" @keydown="input()" />value:{{value}}</div>`,
      model: {
        prop: 'value',
        event: 'input'
      },
      props: ['value','val'],
      methods: {
        input(){
          this.value+=this.val
          this.$emit('input',this.value)
        }
      }
    })
    new Vue({
      el: '#app',
      created () {
        this.$root.$on('add',(val)=>{
          console.log(val)
          this.boxx = val+1
        })
      },
      data () {
        return {
          type: 'text',
          num: '',
          btn:'ddd',
          boxx: 123
        }
      }
    })
  </script>
</body>
</html>

```
### 问题
- 为什么要在自定义组件中使用v-model
**在回答问题之前，我们想理解一下为什么要在表单中使用v-model？**  
> 在我的理解之中，表单中使用v-model指令是为了进行数据的双向绑定。  
**那是为了进行哪两种数据的绑定**  
> 废话，当然是JS与视图中的数据啦。  
eg:
```
// html
<input v-model="val" />

// script
new Vue({
  data () {
    return {
      val: 'hello world'
    }
  }
})
```
上述代码中v-model指令在默认情况下所做的事情就是，将val绑定到input的value上，当input触发input事件时，将input的value赋值给val  
在这个过程中，其实所采用的机制也是eventBus，只是当v-model指令在input表单上使用时，vue封装了自动处理双向绑定的过程。  
**回到我们的原问题：为什么要在自定义组件中使用v-model?**  
> 当然是因为懒啦。为了更少的写代码，为了更高的逼格。(父子之间的数据双向绑定)  
v-model的初衷就是为了表单的数据双向绑定。那么我们使用v-model时，也要符合这个初衷。

- 什么场景适合在自定义组件中使用v-model
**在需要双向数据绑定，或需要修改默认绑定prop与event时**
> 写着写着突然发现好像有很好的方式实现父子之间的双向数据绑定。。。。  
感觉瞬间日了狗了。  

>可以使用 [.sync](https://cn.vuejs.org/v2/guide/components.html#sync-%E4%BF%AE%E9%A5%B0%E7%AC%A6)修饰符达到双向数据绑定的目的。 在子组件中需要进行prop双向绑定时，显示的触发 this.$emit('update:foo', newValue)
eg：
```
<comp :foo.sync="bar"></comp>
// .sync 修饰符会被扩展为
<comp :foo="bar" @update:foo="val => bar = val"></comp>

// 当子组件需要更新foo的值时，需要显示触发一个更新事件。(其中原理依然是eventBus 只是vue已经帮你部分实现了，只需要触发即可)
this.$emit('update:foo', newValue)

**虽然在vue 2.3版本后，又重新将.sync重新引入，实现父子组件上prop的双向绑定**，但是自定义组件上使用v-model，实现双向绑定也许会有一定意义。
```

- 这样使用的好处
> 别问我 ， 我不知道 。
- 使用姿势？
```
// html
  <div id="app">
   <box v-model="num"></box>
   <btn v-model="num"></btn>
  </div>
   // js
   Vue.component('btn', {
    template: `<div><button @click="add()">+</button>{{val}}</div>`,
    model:{
      prop: 'val',
      event: 'change'
    },
    props: ['val'],
    methods: {
      add () {
        this.$emit('change', this.val) 
      }
    }
  })
   Vue.component('box', {
    template: `<div><button @click="add()">+</button>{{val}}</div>`,
    model:{
      prop: 'val',
      event: 'input'
    },
    props: ['val'],
    methods: {
      add () {
        this.$emit('input', this.val) 
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
```
> 分析：  
model属性中的prop与event  
prop为v-model所被扩展后的 prop，event为vue已经写好的监听事件。  
> vue已有的event有 input change 等 我不知道的 。

- .sync 和 v-model  与 使用eventBus 实现prop双向绑定 对比
> 其实三者原理都基于与eventBus实现  
但是.sync和v-model 不需要在写对事件的监听。 因为vue已经对 update change input 事件的监听。  
**若采用eventBus实现，则需要写事件的监听与触发(注意：事件的监听与触发需要在同一个实例上才能被触发)**
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <script src="https://cdn.jsdelivr.net/npm/vue "></script>
  <style>
    #app div {
      margin: 20px 0;
    }
    .red{
    	width: 50px;
    	height:50px;
		background-color: red;
    }
  </style>
  <title>Document</title>
</head>
<body>
  <div id="app">
    <div>
      <btn v-model="btn"></btn>
    {{btn}}
    </div>
    <div>
      <box :value.sync="box"></box>
      {{box}}
    </div>
    <div :class="[active?'red':'']"></div>
  </div>
  <script>
    Vue.component('box', {
      template: `<div><button @click="add()">+</button></div>`,
      model: {
        prop: 'value',
        event: 'input'
      },
      props: ['value'],
      methods: {
        add () {
          this.value++
          console.log(this.value)
        }
      },
      watch: {
        value (val,oldVal) {
          this.$emit('update:value',val)
        }
      }
    })
    Vue.component('btn', {
      template: `<div><input v-model="value"/></div>`,
      model: {
        prop: 'value',
        event: 'input'
      },
      props: ['value'],
      // methods: {
      //   input(){
      //     this.val++
      //     console.log(this.val)
      //     this.$emit('input',this.val)
      //   }
      watch: {
        value (val,oldVal) {
          this.$emit('input',val)
        }
      }
    })
    new Vue({
      el: '#app',
      created () {
        this.$root.$on('add',(val)=>{
          console.log(val)
          this.boxx = val+1
        })
      },
      data () {
        return {
          btn:'123',
          box:'123',
          active:true
        }
      }
    })
  </script>
</body>
</html>
```
