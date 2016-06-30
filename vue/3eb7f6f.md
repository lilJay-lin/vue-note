#3eb7f6f

1. 在`seed`上增加两个方法

    1. `dump`方法，获取bindings属性的备份
    
    2. `destroy`方法，移除所有绑定指令的节点
    
2. 修复filter值被重写问题（只能解决值传递，对象传递依旧会有问题，占坑，看看后面怎么解决的）
    
    ```javascript
    
        /* 旧代码：value作用域在最外层，value重复 */
        set: function (value) {
            binding.value = value
            binding.directives.forEach(function (directive) {
                if (value && directive.filters) {
                    //执行filters
                    value = applyFilters(value, directive)
                }
            }
        }
        /*改动代码*/
        set: function (value) {
            binding.value = value
            binding.directives.forEach(function (directive) {
                var filteredValue = value
                if (value && directive.filters) {
                    filteredValue = applyFilters(value, directive)
                }
                directive.update(
                    directive.el,
                    filteredValue,
                    directive.argument,
                    directive,
                    seed
                )
            })
        }
    ```
