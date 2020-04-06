```
1. 	尽量不要用window.xxx的全局变量，可以用
	比如：
		const PlayerData = function() {
    		const that = {};
    		that.uniqueID = "1" + getRandomStr(6);
    		return that;
		};
		export default PlayerData;

	或者再cocoscreator里，因为cc是默认全局变量，所以可以用用cc.xx = {}  cc.xx.yy = '';

2. 	0.1+0.2 = 0.30000000000004-->违背了数学的基本常识，这种情况经常用(1+2)/3 = 0.3
	因为js不能很好的处理十进制的小数，但是可以很好的处理整数的运算

3.  检测值类型：
		function type(o) {
			return (o === null)? 'null': (typeof o);
		}

4.  Number.parseInt(xx, 10); // 最好加上进制数
	它的规则是：查看位置0的字符，如果不是有效数字则返回NaN，如果是有效数字，则看第二位，依次类推，直到遇到非数字字符
	parseInt(123.45)-->123
	parseInt(010)-->0开头的则作为8进制数字处理
	parseInt(0x10)-->0x开头的则作为16进制数字处理

5.  null：表示对象的引用为空或者不存在
	undefined：对象中没有想获取的属性；函数没有return；函数的实参个数少于形参个数，其余形参的值为undefined；

6.  逗号运算符：
		var a = (1,2,3,4)--->a = 4
		a = 1,2,3,4 --->a = 1
		var a = 1,2,3,4 --->a = null

7.  ****最好把hasOwnProperty当做一个函数来用，用来判断onject是否有某个key

8.  不要使用类型构造器 new Function   new Array  new String等等

9.  不要使用new运算符

10.	if判断语句最好使用 if(1===a) 这种形式

11.	循环结构中，需要动态改变循环变量时候最好使用while，对于静态的循环变量则用for

12. 迭代：待补充

13. 使用制表：
		例如：计算factorial(6)   factorial(7)   factorial(8) 用常规递归的话做的重复工作太多
		用：
			function factorial(n) {
				if (factorial.cache) {
					factorial.cache = {
						'0':1,
						'1':1
					}
				}
				if (factorial.cache.hasOwnProperty(n)) {
					factorial.cache[n] = n * factorial(n - 1);
				}
				return factorial.cache[n];
			}

14. 获取字节长度：
		String.prototype.lengthB = function() {
			let b = 0, l = this.length;
			if (l) {
				for(let i = 0; i < l; i++) {
					if (this.charCodeAt(i) > 255) {
						b += 2;
					} else {
						b += 1;
					}
				}
				return b;
			} else {
				return 0;
			}
		}

15. 大量字符串连接尽量不要用+=

16. ****检测是否为数组：
		let isArray = function(value) {
			return Object.prototype.toString.apply(value) === '[Object Array]';
		}

17. 构建二维数组：
	Array.prototype = function(m, n, value) {
		let a, mat = [];
		for(let i = 0; i < m; i++) {
			a = [];
			for(let j = 0; j < n; j++) {
				a[j] = value;
			}
			mat[i] = a;
		}
		return mat;
	}

18. 奇葩特性，实际并没有用，看到不要发懵就好
	js中的数组下标可以是任何数据类型，任何表达式
	let a = [];
	a[-1] = 1;
	console.log(a.length)-->0 // 数组长度没变
	console.log(a[-1])--->1  //奇葩的有值
	a[true] = 1;
	console.log(a[true])--->1 // 奇葩的有值


19. 通过Function扩展函数：
		Function.prototype.method = function(name, func) {
			if (!this.prototype[name]) {
				this.prototype[name] = func;
				return this;
			}
		}
	利用上面的函数可以扩展一些函数，例如提取数字的整数部分：
		Number.method('integer', function() {
			return Math[this < 0? 'ceil': 'floor'](this);
		});
	(以前没注意的一个知识点：
		Math.ceil()：根据“ceil”的字面意思“天花板”去理解；
		例如：
		Math.ceil(11.46)=Math.ceil(11.68)=Math.ceil(11.5)=12
		Math.ceil(-11.46)=Math.ceil(-11.68)=Math.ceil(-11.5)=-11
		 
		Math.floor()：根据“floor”的字面意思“地板”去理解；
		例如：
		Math.floor(11.46)=Math.floor(11.68)=Math.floor(11.5)=11
		Math.floor(-11.46)=Math.floor(-11.68)=Math.floor(-11.5)=-12)

	// 去除字符串两边的空格
	String.method('trim', function() {
		return this.replace(/^\s+|\s+$/g, '');
	});

20. 函数的调用和引用：
		引用的例子：
			function f() {
				var x = 5;
				return x;
			}
			var a = f;
			var b = f;
			console.log(a === b) // true
			因为函数引用的本质是，多个变量存储的相同的入口指针

		调用的例子：
			function f() {
				var x = 5;
				return function() {
					return x;
				}
			var a = f();
			var b = f();
			console.log(a === b) // false，虽然都是相同的闭包，但是他们存储在不同的变量中，即地址指针是不同的

			function f() {
				var x = 5;
				return x;
			}
			var a = f();
			var b = f();
			console.log(a === b) // true
			函数调用时执行该函数，，比较的是值，而不是入口指针

21. 使用函数来记录先前操作的结果，避免无谓的运算。
	例如：
		let fibonacci = (function() {
			let memo = [0, 1];
			let fib = function(n) {
				let result = memo[n];
				if (typeof result !== 'number') {
					result = fib(n - 1) + fib(n - 2);
					memo[n] = result;
				}
				return result;
			}
			return fib;
		}());

22. 链式语法：很多的函数实现是没有返回值的，即返回undefined，如果想让这些方法返回this，则需要用链式语法。
			这样就可以连续调用同一个对象的很多方法。
			例如：String.method('trim',function(){
					return this.replace(/^\s+|\s+$/g,'');
				 });
				String.method('writeln',function(){
					document.writeln(this);
					return this;
				});
				String.method('alert',function(){
					window.alert(this);
					return this;
				});
				var str="abc";
				str.trim().writeln().alert();

23. 分支函数：在判断浏览器兼容性的问题时，常常用这个方法，通过下面的方法可以看出，可以声明数个不同名称的对象，但是这些对象里都有一个名称相同的方法。
	例如：；let XHR = function() {
				let standard = {
					createXHR: function() {
						return new method1();
					}
				}
				let new1 =  {
					createXHR: function() {
						return new method2();
					}
				}
				let new2 =  {
					createXHR: function() {
						return new method3();
					}
				}
			}
			if (standard.createXHR()) {
				return standard;
			} else {
				if(new1.createXHR()) {
					return new1;
				} else {
					return new2;
				}
			}
24. 
