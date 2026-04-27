# 类型
## 类型系统
js中的类型分为**原始类型**和**引用类型**。
- 原始类型
	- Number
		- 底层使用[[IEEE 754浮点数]]储存
		- 两种特殊值
			- NaN：指数位全1，尾数位非全0
			- Infinity：指数位全1，尾数位全0
	- String
	- Boolean
	- Undefined
	- Null
	- BigInt
	- Symbol
- 引用类型——对象
	- 一般对象
	- 集合对象：`Array, Set, Map, WeakSet, WeakMap`
	- 函数：`Function`
	- 其它：`RegExp, Date, Error`等

## 基本类型
### BigInt
- 作用：解决`Number`精度丢失问题
- 创建方式
	- 整数字面量后加`n`：`10000n`
	- `BigInt()`函数：`BitInt(10000)`
- 底层使用类数组结构，将大整数分成若干块，每块32或64位，按顺序存放在堆内存中
- 与`Number`的运算
	- 不可进行算术运算：`10n + 2 // 报错`
	- 可以进行比较运算：`10n > 6 // true`
- `BigInt`间的除法运算：按照整数触发，截断小数部分：`5n / 2n === 2n`

### Symbol
- 作用：生成唯一标识符
- 创建方式
	- 普通`Symbol`：`const s = Symbol("foo")`‘
	- 全局`Symbol`：`const s = Symbol.for("foo")`
		- 使用`Symbol.keyFor("foo")`获取到之前注册过的`Symbol`

## 内存分配
- 分配
	- 基本类型：分配在栈上
	- 引用类型：对象内容存放在堆上，并在栈中存入该对象的指针
- 拷贝
	- 基本类型：值拷贝
	- 引用类型：引用拷贝（浅拷贝）

> [!note] **基本类型的变量一定存储在栈上吗？**
> 不一定，当变量发生逃逸（如闭包内引用外部变量）时，js引擎会将该变量提升到堆上。

## `typeof`和`instanceof`
### `typeof`
语法：`typeof val`
返回值类型：字符串

| 类型        | 返回值       |
| --------- | --------- |
| Number    | number    |
| String    | string    |
| Boolean   | boolean   |
| Undefined | undefined |
| Null      | object    |
| BigInt    | bigint    |
| Symbol    | symbol    |
| Object    | object    |
| Function  | function  |

特殊情况：
- `typeof null`的返回值是`"object"`
- `typeof Function`的返回值是`"function"`而不是`"object"`

> [!note] 为什么`typeof null === "object"`？
> 这是一个历史遗留问题。
> 在js的早期版本中，一个值由类型标签（低3位）和实际值组成。`Object`类型的类型标签为`000`。而`null`的含义是空指针，在大多数平台中，空指针的表示为`0x00`，低3位为`000`，因此在`typeof`中被误判成`Object`。


### `instanceof`
语法：`A instanceof B`
返回值类型：`Boolean`
作用：判断`B`是否在`A`的原型链中。

``` js
function instanceof(A, B) {
	// 如果A是基本类型，直接返回false
	if (
		A === null || 
		!(typeof A === "object" || typeof A === "function")
	)
		return false;

	// 沿A的原型链向上查找
	let p = Object.getPrototypeOf(A);
	while (p) {
		if (p === B.prototype) return true;
		p = Object.getPrototypeOf(p);
	}
	
	return false;
}
```

# 作用域
## 词法作用域
在javascript中，采用词法作用域机制，一个变量的作用域在书写时就已确定。
``` js
var a = 10;
function fn1() {
	console.log(a);
}

function fn2() {
	var a = 20;
	fn1();
}

fn2(); // 10
```

## 作用域类型
javascript中有三种作用域：
- 全局作用域：在函数外声明的变量为全局作用域，在任何地方均可访问
- 函数作用域：在函数内声明的变量为函数作用域，只有在该函数内可访问
- 块级作用域（ES6新增）：在`{}`内使用`let`或`const`声明的变量为块级作用域，只有在该代码块内可访问

## 作用域链
当引用一个变量时，会从当前作用域逐层向外查找，直到全局作用域。
```js
var a = 10;

function fn1() {
	function fn2() {
		console.log(a);
	}
}

fn1(); // 10
// fn2()函数作用域 -> fn1()函数作用域 -> 全局作用域
```

## `var/let/const`
1. 作用域
	- `var`是函数作用域或全局作用域的
	- `let/const`是块级作用域的
2. 变量提升
	- `var`声明的变量会被提升到函数作用域顶部，在声明并初始化之前，该变量的值为`undefined`
	- `let/const`声明的变量不会被提升，从块级作用域开始到声明语句完成之间，如果引用该变量会直接报错。这段区域被称为暂时性死区（TDZ）
3. 重复声明
	- `var`声明的变量可以重复声明
	- `let/const`重复声明会报错
4. 可修改性
	- `var/let`声明的变量可以修改
	- `const`声明的变量不可以修改
		- 在`const`声明的对象变量中，`const`只能保证指针指向不变，不能保证指针指向的内容本身不变

> [!note] 最佳实践
> 优先使用`const`，需要修改变量值时使用`let`，不使用`var`。

