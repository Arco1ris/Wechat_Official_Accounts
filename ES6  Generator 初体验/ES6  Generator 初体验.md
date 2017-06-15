### ES6  Generator 初体验

听说 ES6 的 Generator 是一个很神奇的函数，所以去了解了一下。 因为它不同于以往的寻常函数，但是带来的体验却非常好 。这里首先讲了 Generator 是什么，分割线后面用了一个例子来说明 Generator 到底好在哪里 ~ （可以选择性阅读~ ）

Generator 是一种异步编程解决方案，不明白？往下看 ~ 

Generator 到底是什么？官方文档是这样说的：“Generators are functions which can be exited and later re-entered. Their context (variable bindings) will be saved across re-entrances.”

意思就是 Generator 函数内部的执行是可以暂停的。你能够执行到一半退出函数，同时执行到当前的上下文环境（包括变量等等）全部被保存下来，如果我们稍后又进来继续执行下一段，此时是在上次状态的基础上继续往下执行。感觉和别的函数很不一样，如何去理解呢？

直接上代码：

``` javascript
var myGen = function*(){  // 定义一个 Generator 函数
  var one = yield 1; 
  var two = yield 2;
  var three = yield 3;
  console.log(one,two,three);
}
var gen = myGen();   
console.log(gen.next());  //第一次执行函数，输出 {value:1,done:false}
console.log(gen.next());  //第二次执行函数，输出 {value:2,done:false}
console.log(gen.next())； //第三次执行函数，输出 {value:3,done:false}
console.log(gen.next());  //第四次执行函数，输出{value:undefined,done:true}
```

可以看到 function 后面加了一个 * 号，那是特有的 Generator 的写法，**function*** 就表示我定义了一个 Generator function。内部还有三个 yield 。

那么上面那段函数是如何执行的呢，一步一步来：

1. 定义一个  Generator 函数 —— var myGen = function*(){…} ；
   
2. var gen = myGen()  进行赋值，接下来想要执行函数需要调用 next()，因为 Generator 函数不会自己执行，next() 可以 return 一个对象，包含两个属性，一个是输出的值（yielded value），还有一个是 done property，表示这个  Generator 函数是否已经输出了最后一个值（yielded its last value）了；
   
3. 调用 next() 后，执行函数内部第一条语句 var one = yield 1，由于这条函数执行顺序是自右向左，所以先执行  yield 1，yield 1 这里的作用类似  return 1，返回一个 1 ，同时暂停了函数的执行，它后面的语句并不会继续执行下去。所以第一句 console.log(gen.next()) 输出 1。
   
4. 以此类推，最后的 console.log(gen.next()) 结果为 {value:undefined,done:true}，因为函数已经没有任何 yield，没有 return 任何值回来，done 为  true 表示这个 Generator 函数执行结束了。
   
5. myGen 函数内的 console.log(one,two,three) 会输出什么呢？ 1,2,3 ？猜错了！其实这里输出的是 undefined,undefined,undefined。因为 yield 在 var one = 和 1 中间，所以 1 并没有赋值到变量 one 上面。
   
   下面加一点好玩的东西：把上面所有的 console.log() 改成下面这个样子。
   
   ``` javascript
   console.log(gen.next());
   console.log(gen.next(4));
   console.log(gen.next(5));
   console.log(gen.next('a'));
   ```
   
   这时， console.log(one,two,three) 输出的就是 4,5,a。
   
   WHY？？  刚刚说到 var one = yield 1;的执行顺序是自右向左的，执行到 yield 1就暂停了，所以并没有给 one 赋值，所以执行 console.log(gen.next(4)) 语句的时候，把参数 4 赋值给了 one ，然后继续执行 yield 2，以此类推...

然而这看起来好像并没有什么卵用？别急，继续往下看 ~ 

———————————我是分割线—————————————

首先我们知道平常经常会遇到这样一种情况，就是需要写嵌套的 AJAX，特别是比较复杂的项目里面，AJAX 一层层嵌套（ 会存在多层回调嵌套，叫做 Pyramid of Doom），体验非常糟糕。

比如下面这样子的：

``` javascript
$.ajax({          // 第一个AJAX
  type:'GET',
  url:'booklist.json',
  success:function(booklist){
    	console.log(booklist);
    	$.ajax({   // 第二个AJAX
          type:'GET',
          url:'book.json?id='+booklist.id,
          success:function(book){
            	console.log(book);
  				$.ajax({    //第三个AJAX
                  type:'GET',
                  url:'comments.json?id='+book[0].id,
                  success:function(comments){
						console.log(comments);
					},
                  error:function(xhr,status,error){
  						//处理一些东西
					}
				});
			},
          error:function(xhr,status,error){
  				//处理一些东西
			}
		})；
	}，
  error:function(xhr,status,error){
  		//处理一些东西
	}
})
```

三个AJAX嵌套在一起，虽然被简化了（一些数据处理直接用 console.log() 替代），但是还是看起来比较乱，一乱起来中午吃个饭回来都找不到上次写到哪里了，然而实际业务中要做的处理也绝对比这多很多。

这里先用一种比较普通的方式来处理上面那大段代码：

``` javascript
$.ajax({
  type:'GET',
  url:'booklist.json',
  success:getbook,
  error:handleError
});
function getbook(booklist){
  console.log(booklist);
  $.ajax({
    type:'GET',
    url:'book.json?id='+booklist.id,
    success:getComments,
    error:handleError
	});
}
function getComments(book){
  console.log(book);
  $.ajax({
    type:'GET',
    url:'comments.json?id='+book[0].id,
    success:function(comments){
		console.log(comments);
		},
    error:handleError
	});
}
function handleError(xhr,status,error){
  //处理一些东西
}
```

上面的代码封装了三个 function，看起来整齐一些了，并且所有的 error 处理都通过调用自己封装好的 handleError 去做，然而我们的代码量并没有减少多少。

下面用 Promise 对代码进行处理：

``` javascript
$.get(booklist.json).then(function(booklist){
  console.log(booklist);
  return $.get('book.json?id='+booklist.id);
}).then(function(book){
  console.log(book);
  return $.get('comments.json?id='+book[0].id)
}).then(function(comments){
  console.log(comments);
},handleError);

function handleError(xhr,status,error){
	//处理一些东西
}
```

Promise 通过 .then() 将函数串联在一起，它采用了同步的方式去处理异步的问题，去除了一层层的回调嵌套，error 处理只需要最后调用一次 handleError 就够了，似乎已经很好了，不过这里还有更好的方法——Generator。

下面是用 Generator 处理：

``` javascript
Promise.coroutine(function*(){
  var booklist = yield $.get('booklist.json');
  console.log(booklist);
  var book = yield $.get('book.json?id='+booklist.id);
	console.log(book);
  var comments = yield $.get('comments.json?id='+book[0].id);
	console.log(book);
})().catch(function(errs){
  //处理一些东西
})
```

对比第一段代码，瞬间神清气爽了！Promise.coroutine 是专门用来给 Generator 用的，作用就相当于最开始的那个带上参数的 next()。我们不需要去把回调函数嵌套在一起，或者串联在一起就能够得到想要的结果，是不是非常赞~？！！ꉂ ೭(˵¯̴͒ꇴ¯̴͒˵)౨”   

References：

[https://www.youtube.com/watch?v=QO07THdLWQo](https://www.youtube.com/watch?v=QO07THdLWQo)

[https://www.youtube.com/watch?v=obaSQBBWZLk](https://www.youtube.com/watch?annotation_id=annotation_1449687199&feature=iv&list=UUVTlvUkGslCV_h-nSAId8Sw&src_vid=QO07THdLWQo&v=obaSQBBWZLk)

