```
---------------讲的忒细，忒冷门------------------------

1. 判断0和-0，不能用===，要用下面的函数
	判断是否是NaN的话，只能用Number.isNaN()

	function isNegZero(n) {
		n = Number(n);
		return (n === 0) && (1 / n === -Infinity)
	}

2.  简单值(基本类型值)：通过值复制的方式来传递
	复合值(对象)：数组和对象通过引用复制的方式(指针)来传递

	js中引用指向的是值，比如一个值有10个引用，那么这些引用指向的都是同一个值，他们相互之间没有引用/指向关系

	例如：
		let c = [1, 2, 3];
		let d = c;
		d.push(4);
		c;--->[1,2,3,4]
		d;--->[1,2,3,4]

	又比如：
		let c = [1, 2, 3]
		let d = c;
		d = [4,5,6];
		c;--->[1,2,3]
		d;--->[4,5,6]


3.  不能这样赋值：let a = b = 1;
	相当于声明来了一个全局变量b

4.  break  continue新用法： 不过不建议使用
	
	function foo() {
		for(let i = 0; i < 4; i++) {
			for(let j = 0; j < 4; j++) {
				if (j === i) {
					continue foo; //跳入foo的下一个循环，也就是i的下一个循环
				}
				if ((j * i) % 2 === 1) {
					continue;
				}
				console.log(i, j)
			}
		}
	}
	// 1  0
	// 2  0
	// 2  1
	// 3  0
	// 3  2

	function foo() {
		for(let i = 0; i < 4; i++) {
			for(let j = 0; j < 4; j++) {
				if ((i * j) >= 3) {
					console.log('Stop!!!!', i, j);
					break foo;
				}
				console.log(i, j);
			}
		}
	}
	//  0 0
		0 1
		0 2
		0 3
		1 0
		1 1
		1 2
		Stop!!! 1 3

5.  es6中，如果函数参数被省略或者定义undefined，则取该值的默认值
	 function foo(a = 42, b = a + 1) {
	 	console.log(a, b);
	 }
	 foo()
	 foo(undefined) /////42  43
	 foo(null)  // null  1 --->null被强制转为0
