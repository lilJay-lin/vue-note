# 23a2bde: 重构

1. 增加控制器概念, 逻辑代码划分作用域
2. 变更指令书写方式

## 主要修改点

1. 指令书写方式： 参数写到属性值里，使用*:*做分隔，事件指令可以一次性绑定多个事件，多个事件使用*,*分隔

    ````html
        
		<div id="app" sd-controller="TodoList">
            <p sd-text="msg | capitalize" sd-on="click:changeMessage"></p>
            <p sd-text="msg | uppercase"></p>
            <p sd-on="click:remove">bye</p>
            <p sd-text="total | money"></p>
            <p sd-class="red:error" sd-show="error">Error</p>
            <ul sd-show="todos">
            	<li class="todo"
                    sd-controller="Todo"
                    sd-each="todo:todos"
                    sd-class="done:todo.done"
                    sd-on="click:todo.toggle"
                    sd-text="todo.title"
                ></li>
            </ul>
        </div>
    ```

2. 控制器**Controller**

    ````javascript
    
        //main.js
        
        1. 新增Controller， 返回Seed构造函数，可访问控制器内的属性，实例化之后对scope做初始赋值
            Seed.controller (id, extensions) => {
               return Seed.extend(extensions)  //返回新包装的Seed构造函数，在实例scope上添加extensions赋值
            }
        
        2. Seed.bootstrap 实例化
            Seed = 是否是控制器（属性值sd-controller=ctrlid) ？ Controller[ctrlid] : Seed
            new Seed(el, data, options)
        
        //seed.js
        
        3. 实例化做数据绑定
           因为存在不同的Controller（即不同属性作用域），所以数据需要绑定在正确的作用域上
           示例：
           sd-controller="TodoList"  
                sd-controller="Todo"
                    sd-each="todo:todos"
                    sd-class="done:todo.done"
                    sd-on="click:todo.toggle"
            
           TodoList下的指令构造函数 = controller[TodoList]
           sd-each 指令节点存在sd-controller="Todo", 编译的todo模板实例化函数使用controller[Todo]
                sd-class="done:todo.done", sd-on="click:todo.toggle" 实例化：
                new Ctrl(node, data, {
                    eachPrefixRE: this.prefixRE,
                    parentScope: this.seed.scope
                })
                //TODO:each 的指令监听绑定在parentScope, 数组的多个单元重复在parentScope上做监听绑定，如何处理覆盖问题 
    ````