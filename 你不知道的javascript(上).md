```
前面关于编译原理、eval、with部分并不想看。

1. ReferenceError 同作用域判别失败相关，而 TypeError 则代表作用域判别成功了，但是对
	结果的操作是非法或不合理的。

	词法作用域：由书写代码时或函数声明的位置来决定的
				作用域查找会在找到第一个匹配的标识符时停止。
				作用域查找始终从运行时所处的最内部作用域开始，逐级向外或者说向上进行，直到遇见第一个匹配的标识符为止。
	函数作用域：属于这个函数的全部变量都可以在整个函数的范围内使用及复用（事实上在嵌套的作用域中也可以使用）

		//始终给函数表达式命名是一个最佳实践
		setTimeout( function timeoutHandler() { // <-- 名字
			console.log( "I waited 1 second!" );
		}, 1000 );

		IIFE(立即执行函数)：函数被包含在一对 () 括号内部，通过在末尾加上另外一个()可以立即执行这个函数，这样函数成为了一个表达式
		(function(参数){ .. }(参数)) 等价于 (function(参数){ .. })(参数);

		函数声明和函数表达式之间最重要的区别是它们的名称标识符将会绑定在何处。
		函数声明：被绑定在所在作用域中，可以直接通过foo() 来调用它。
		函数表达式：foo 被绑定在函数表达式自身的函数中而不是所在作用域中。

		常用的用法：
			var a = 2;
			(function IIFE( global ) {
				var a = 3;
				console.log( a ); // 3
				console.log( global.a ); // 2
			})( window );

			console.log( a ); // 2
			

			(function IIFE( def ) {
				def( window );
			})(function def( global ) {
				var a = 3;
				console.log( a ); // 3
				console.log( global.a ); // 2
			});

	提升：
		a = 2;
		var a;
		console.log( a ); //2

		console.log( a ); // undefined
		var a = 2;

		解释：运行js代码时：编译器首先进行编译，再进入等待执行阶段
			 编译阶段时：会找到所有的(变量和函数)
			 声明，并用作用域将他们关联起来，这里要注意的是函数和变量声明会被提升，但是函数表达式不会被提升
			 函数声明首先被提升，变量声明后被提升，相同的名字的声明后者会覆盖前者，这个提升不受if判断的影响。

			 等待执行阶段时：赋值声明会留在原地等待执行

			 对于函数表达式：一定是要先定义，再使用

			 所以第一个代码相当于：
			 	var a;
			 	a = 2;
				console.log( a ); //2
			第二段代码相当于：
				var a
				console.log( a ); // undefined
				a = 2;

	(重点！！！)
	闭包：在任何同步或异步的任务中，只要使用了回调函数，实际上就是在使用闭包，他们都持有对原始定义作用域的引用，可以记住并访问所在的词法作用域
		即使函数是在当前词法作用域之外执行。

		比如传递的是函数类型的值
			function foo() {
				var a = 2;
				function baz() {
					console.log( a ); // 2
				}
				bar( baz );
			}
			function bar(fn) {
				fn(); //闭包！
			}

			function foo() {
				var a = 2;
				function bar() {
					console.log( a );
				}
				return bar;
			}
			var baz = foo();
			baz(); // 2
		上面就是一个闭包的例子，当执行完foo()时，通常来说 foo里的变量都应该释放了，但是return bar阻止了这个释放，其内部作用域
		依然存在，因此没有被回收

	典型的例子：
		for (var i=1; i<=5; i++) {
			(function() {
				setTimeout( function timer() {
					console.log( i );
				}, i*1000 );
			})();
		}---->66666

		for (var i=1; i<=5; i++) {
			(function(j) {
				setTimeout( function timer() {
					console.log( j );
				}, i*1000 );
			})(i);
		}---->12345


	模块是闭包的一个重要应用
		function CoolModule() {
			var something = "cool";
			var another = [1, 2, 3];
			function doSomething() {
				console.log( something );
			}
			function doAnother() {
				console.log( another.join( " ! " ) );
			}
			return {
				doSomething: doSomething,
				doAnother: doAnother
			};
		}
		var foo = CoolModule();
		foo.doSomething(); // cool
		foo.doAnother(); // 1 ! 2 ! 3

		模块模式的两个必要条件：
		1  必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
		2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并
			且可以访问或者修改私有的状态。

			var foo = (function CoolModule(id) {
			function change() {
				// 修改公共 API
				publicAPI.identify = identify2;
			}
			function identify1() {
				console.log( id );
			}
			function identify2() {
				console.log( id.toUpperCase() );
			}
			var publicAPI = {
				change: change,
				identify: identify1
			};
				return publicAPI;
			})( "foo module" );
			foo.identify(); // foo module
			foo.change();
			foo.identify(); // FOO MODULE

2. this相关：重点！！！

	// 一道经典题
	function foo() {
		var a = 2;
		this.bar();
	}
	function bar() {
		console.log( this.a );
	}
	foo(); // this.bar is not a function

	this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用(不是声明位置)。
	最关键的是要找出调用栈，即最后一次调用的位置

	注：严格模式下this默认指向undefined，非严格模式下this指向window，不要混用这两个模式

	有4条规则：
		1 默认绑定：严格模式下绑定到undefined，非严格模式绑定到window

		2 隐式绑定：使用对象上下文来引用函数时，函数里的this就会绑定到这个对象。只有最顶层或者说最后一层会影响调用位置
		  	隐式丢失：被隐式绑定的函数会丢失绑定对象，即this会应用默认绑定
					例子：比如

							function foo() {
								console.log( this.a );
							}
							var obj = {
								a: 2,
								foo: foo
							};
							var bar = obj.foo; // 函数别名！
							var a = "oops, global"; // a 是全局对象的属性
							bar(); // "oops, global"

						又比如：在传入回调函数时：
							function foo() {
								console.log( this.a );
							}
							function doFoo(fn) {
								// fn 其实引用的是 foo
								fn(); // <-- 调用位置！
							}
							var obj = {
								a: 2,
								foo: foo
							};
							var a = "oops, global"; // a 是全局对象的属性
							doFoo( obj.foo ); // "oops, global"

						又比如：
							function foo() {
								console.log( this.a );
							}
							var obj = {
								a: 2,
								foo: foo
							};
							var a = "oops, global"; // a 是全局对象的属性
							setTimeout( obj.foo, 100 ); // "oops, global"
							
							类似于这样
							function setTimeout(fn,delay) {
								// 等待 delay 毫秒
								fn(); // <-- 调用位置！
							}

		3 显式绑定： call apply:两者的第一个参数是一个对象，它们会把这个对象绑定到this ，接着在调用函数时指定这个this，不可变，
					如果参数传入了null或undefined，则为默认绑定。
					es5中提供了bind函数

					call apply bind的异同：
						相同：都是用来改变函数的this对象的指向的；第一个参数都是this要指向的对象；都可以利用后续参数传参。
								如果把null货undefined作为this的绑定对象，则无效，使用的是默认绑定规则

						不同：call apply都是对函数的直接调用；bind返回的仍然是函数，要执行的话需要加(param1 ... paramn)
							  call的参数可以有多个，apply的参数必须只有一个，如果要传多个，可以用数组等进行封装
		4 new 不推荐使用

		上述4种优先级的顺序是：new  >>>>  call apply bind  >>>>>  是否在某个上下文的对象中调用，即隐式绑定 >>>>> 默认

		5 箭头函数：
			es6中的箭头函数并不在上述规则以内，一旦绑定就不能修改，是继承外层函数调用的this绑定
			箭头函数需要注意的是：函数体内的this对象，就是定义生效时所在的对象，而不是使用生效时所在的对象。
							实际原因是箭头函数根本没有自己的this，导致内部的this就是外层代码块的this。

			有外层函数的this就是箭头函数的this，无外层函数则指向window，不用管其他
			也可以这么说：箭头函数的this是继承父执行上下文里面的this，这里箭头函数的执行上下文是函数f()，所以它就继承了f()的this
    						注意：简单对象（非函数）是没有执行上下文的！


		不过应该在下列情形下尽量不要使用箭头函数：
			1 在对象方法内最好不要使用：
				例子：  this.food = "banana";
						let obj = {
						    food: "strawberry",
						    log: () => {
						        console.log(this.food);
						    }
						};
						obj.log(); // 打印"banana" ：箭头函数自身没有 this 会导致自动继承外层的 this

			2 上下文是可变的，动态的，那么不要使用箭头函数

3.  js对象有一个特殊的内置属性 [[Prototype]]，是对于其他对象的引用
	作用：当试图引用对象的属性时会触发[[Get]] 操作，比如 myObject.a 。对于默认的 [[Get]] 操作来说，第一步是检查对象本身是
		 否有这个属性，如果有的话就使用它。但是如果 a 不在 myObject 中，就需要使用对象的 [[Prototype]] 链了。
		对于默认的 [[Get]] 操作来说，如果无法在对象本身找到需要的属性，就会继续访问对象的 [[Prototype]] 链。
		
