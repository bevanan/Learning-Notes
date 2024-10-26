# Vue3

现在都是用vite创建一个vue3项目





Vue2 是选项式的模式

vue3 是组合模式



<img src="/Users/zhengbufeng/Documents/学习笔记/vue3/vue3.assets/image-20240720103024582.png" alt="image-20240720103024582" style="zoom:33%;" />

<img src="/Users/zhengbufeng/Documents/学习笔记/vue3/vue3.assets/image-20240720103104312.png" alt="image-20240720103104312" style="zoom: 33%;" />

上面两图说明的是 App.vue是组件（也就是根），在components内都是App.vue的子组件（根的枝）

那么main.ts内 就是作用在一下大的entry里面

<img src="/Users/zhengbufeng/Documents/学习笔记/vue3/vue3.assets/image-20240720104631796.png" alt="image-20240720104631796" style="zoom:33%;" />



## 编写一个Person.vue

利用vue2和vue3的两种方式编写

### vue2

那么自定义写一个App.vue

<img src="/Users/zhengbufeng/Documents/学习笔记/vue3/vue3.assets/image-20240720103225473.png" alt="image-20240720103225473" style="zoom:33%;" />

Vue2 的方式

<img src="/Users/zhengbufeng/Documents/学习笔记/vue3/vue3.assets/image-20240720103347496.png" alt="image-20240720103347496" style="zoom:33%;" />

### vue3

用目前vue3的方式 用setup 语法糖改写，改字段

<img src="/Users/zhengbufeng/Documents/学习笔记/vue3/vue3.assets/image-20240720103405233.png" alt="image-20240720103405233" style="zoom:33%;" />

也可以return {name, age}

改方法

<img src="/Users/zhengbufeng/Documents/学习笔记/vue3/vue3.assets/image-20240720103456951.png" alt="image-20240720103456951" style="zoom:33%;" />

所以函数也要返回出去return {name, age, changeName, changeAge, showTel}



```vue
<script lang="ts">
    export default {
        name: 'Person',
        setup(){
            let name = '张三',
            let age = 18,
            let tel = '188888888888'
                    
            function changeName (){
                name = "zhang-san"
            }
            
            return {name, age, changeName}
        }
    }
</script>
```











