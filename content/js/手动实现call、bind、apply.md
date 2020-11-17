# 原生js实现 call
    1、将当前调用的函数变成指定 context 的一个属性由 context 去调用当前函数。
    2、不定参数问题用 eval 去解决 注意 参数为 string 的时候会出现报错的情况。
```javascript
Function.prototype.myCall = function() {
    const args = []
    for(let i = 0; i< arguments.length; i++) {
        args.push(arguments[i])
    }
    /**
     * 没有传入上下文 或者传入的为 null 就认为上下文是 window
     */
    const context = args.shift() || window
    /**
     * 修改原形了链 不把新增的属性添加到 需要绑定的 上下文环境中 、也避免不小心删除了 绑定上下文中的 方法
     * 即 再 context 中 打印 this 不会出现 ___fn 这个函数
     */
    const prototype = Object.getPrototypeOf(context)
    prototype.___fn = this
    const result = eval('context.___fn('+ args +')')
    delete prototype.___fn
    return result
}
```

# 原生js实现 apply
    实现大致思路和 call 大致相同，只是 函数的 arguments 第二个参数存在的话为数组
```javascript
Function.prototype.myApply = function() {

    const args = []
    for(let i = 0; i< arguments.length; i++) {
        args.push(arguments[i])
    }

    /**
     * 没有传入上下文 或者传入的为 null 就认为上下文是 window
     * 即 再 context 中 打印 this 不会出现 ___fn 这个函数
     * 将 args 第一个参数移除之后剩下的应该是一个数组
     */
    const context = args.shift() || window

    /**
     * 修改原形了链 不把新增的属性添加到 需要绑定的 上下文环境中
     */
    const prototype = Object.getPrototypeOf(context)
    prototype.___fn = this
    let result = args[0] ? eval('context.___fn('+ args[0] +')') : context.___fn()
    delete prototype.___fn

    return result
}
```
# 原生js实现 bind
    原生的 bind 对于构造函数是不会生效的
    1、利用闭包 创建一个空的函数 empty 把 this 的原型赋值给 empty 然后。
    2、返回一个函数 fBound 并且改变其原型链为 new empty，用与判断是怎么调用函数的，并且之前函数的原型方法也会保存。
    3、判断调用函数是通过构造调用（this指向fBound）还是普通的调用（this指向window）。
    4、如果是 构造调用 empty.prototype.isPrototypeOf(this) context 为 this 否则 this 为 arguments 的第一个参数
```javascript
Function.prototype.myBind = function() {
    const args = []
    const _self = this
    for(let i = 0; i< arguments.length; i++) {
        args.push(arguments[i])
    }

    const argsContext = args.shift()

    // 中妆函数
    function empty() {}
    empty.prototype = _self.prototype

    function fBound() {
        let context
        /**
         * 构造调用函数 上下文为 this
         * 否则 上下文为 arguments[0]
         */
        if(empty.prototype.isPrototypeOf(this)) {
            context = this
        } else {
            context = argsContext
        }
        const prototype = Object.getPrototypeOf(context)
        prototype.___fn = _self
        try {
            const bindArgs = []
            for(let i = 0; i< arguments.length; i++) {
                // 必须这样 否则 使用构造传入 字符串 的时候会报错
                bindArgs.push('arguments[' + i + ']')
            }
            /**
             * 利用 finally 在 return 之后执行的特性 删除定义的 ___fn
             */
            return args.length ? context.___fn() : eval('context.___fn('+ args.concat(bindArgs) +')')
        } finally {
            delete prototype.___fn
        }
    }

    // 继承之前的方法,并且保存之前的原型的依据
    fBound.prototype = new empty()

    return fBound
}
```
