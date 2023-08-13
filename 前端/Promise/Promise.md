# Promise

Promise是一种异步编程的解决方案，用于处理异步操作并返回结果。

主要作用是解决回调函数嵌套（回调地狱）的问题，使异步操作更加清晰、易于理解和维护。

## Promise中的状态

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    // 三个状态: pending、fulfilled、rejected
    // 可以pending -> fulfilled
    // 可以pending -> rejected
    // 不可以fulfilled -> rejected或者反之
    // 状态只切换一次
    const p1 = new Promise((resolve, reject) => {
      resolve('ok')
      reject('error')
    })
  </script>
</body>
</html>
```

## then方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>

  <script>
    // then: 得到value的成功回调和得到reason的失败回调。
    // 成功的状态: 执行then方法的第一个回调函数
    // 失败的状态: 执行then方法的第二个回调函数
    
    // then方法返回一个新的Promise实例化对象，对象状态取决于回调函数中的内容
    // 1. 如果返回为非Promise实例化对象，则得到一个成功的Promise
    // 2. 如果返回为Promise实例化对象，则Promise实例化对象的状态和结果值将直接影响result常量的状态和结果值
    // 3. 如果抛出异常，则新的Promise实例化对象(result)为失败的Promise

    const p1 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    // 1. 如果返回为非Promise实例化对象，则得到一个成功的Promise
    const result1 = p1.then(value => {
      console.log('then value: ', value)
    }, reason => {
      console.log('then reason: ', reason)
    })

    console.log(result1)

    //*****************//

    const p2 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    // 1. 如果返回为非Promise实例化对象，则得到一个成功的Promise
    const result2 = p2.then(value => {
      return value
    }, reason => {
      return reason
    })

    console.log(result2)

    //*****************//

    const p3 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    // 2. 如果返回为Promise实例化对象，则Promise实例化对象的状态和结果值将直接影响result常量的状态和结果值
    const result3 = p3.then(value => {
      return new Promise((resolve, reject) => {
        resolve('resolve3')
      })
    }, reason => {
      return new Promise((resolve, reject) => {
        reject('reject3')
      })
    })

    console.log(result3)

    //*****************//

    const p4 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    // 2. 如果返回为Promise实例化对象，则Promise实例化对象的状态和结果值将直接影响result常量的状态和结果值
    const result4 = p4.then(value => {
      return new Promise((resolve, reject) => {
        reject('reject4')
      })
    }, reason => {
      return new Promise((resolve, reject) => {
        reject('reject4')
      })
    })
    
    console.log(result4)

    //*****************//

    // 3. 如果抛出异常，则新的Promise实例化对象(result)为失败的Promise
    const p5 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    const result5 = p5.then(value => {
      throw '异常信息'
    }, reason => {
      throw '异常信息'
    })
    
    console.log(result5)
  </script>
</body>
</html>
```

## 链式调用

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  
  <script>
    const p1 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    p1.then(value => {
      console.log('then value: ', value)
    }).then(value => {
    }).then(value => {
      
    })
  </script>
</body>
</html>
```

## all方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>

    const p1 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    const p2 = new Promise((resolve, reject) => {
      // reject("p2 err")
      resolve('hello')
    })

    const p3 = new Promise((resolve, reject) => {
      // reject("p3 err")
      resolve('success')
    })

    // all方法作用: 主要是针对于多个Promise的异步任务的处理
    // 参数: 元素类型是Promise的数组类型
    // 返回值: Promise类型的对象
    //   当数组中所有的Promise对象的状态都是成功的，最终的结果就是成功的Promise，结果值是由每一个Promise的结果值组成的数组
    //   当数组中的Promise对象有一个失败，最终的结果也是失败的Promise，结果值就是失败的这个Promise的结果值
    const result = Promise.all([p1, p2, p3])
    console.log(result)

  </script>
  
</body>
</html>
```

## allSettled方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  
  <script>
    // allSettled方法用来确定一组异步的操作是否都结束了(不管是成功还是失败)
    // 其中包含了fulfilled和rejected两种情况

    function ajax(url) {
      return new Promise((resolve, reject) => {
        let xhr = new XMLHttpRequest()
        xhr.open('get', url, true)
        xhr.send()
        xhr.onreadystatechange = function() {
          if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300) {
              resolve(xhr.responseText)
            } else {
              reject(xhr.responseText)
            }
          }
        }
      })
    }

    Promise.allSettled([
      ajax('http://iwenwiki.com/api/'),
      ajax('')
    ]).then(value => {
      // 过滤成功和失败的两种情况
      let successList = value.filter(item => item.status === 'fulfilled')
      console.log(successList)
      
      let errorList = value.filter(item => item.status === 'rejected')
      console.log(errorList)
    }).catch(reason => {
      console.log(reason)
    })
  </script>
</body>
</html>
```

## any方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  
  <script>
    // Promise下的any方法，只要参数中有一个Promise实例化对象的状态为fulfilled，则整体的结果就为fulfilled，以第一个成功Promise的结果为准
    let p1 = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('ok')
      }, 3000)
    })
    let p2= new Promise((resolve, reject) => {
      setTimeout(() => {
        reject('err')
      }, 1000);
    })
    let p3= new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('success')
      }, 2000);
    })

    let result = Promise.any([p2, p1, p3])
    console.log(result)
  </script>
</body>
</html>
```

## race方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  
  <script>
    // Promise.race方法接收一个Promise实例化对象的数组。哪个对象先从pending状态转为fulfilled或者rejected状态，race返回的Promise将以这个对象的状态和结果为准
    const p1 = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(1)
      }, 2000);
    })
    const p2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        reject(2)
      }, 1000);
    })
    const p3 = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(3)
      }, 3000);
    })

    const result = Promise.race([p1, p2, p3])
    console.log(result)
  </script>
</body>
</html>
```

## reject方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    // Promise.reject方法将始终返回一个失败的Promise对象，无论参数是否为Promise还是其他，最终都返回的是失败Promise
    const p1 = Promise.reject(123)
    console.log(p1)
  </script>
</body>
</html>
```

## resolve方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    // Promise.resolve方法作用: 将一个普通的值转换为Promise类型的数据
    // 两种情况:
    // 1. 当resolve方法参数为非Promise对象，则返回的结果为成功的Promise对象
    // 2. 当resolve方法参数为Promise对象，则返回的Promise对象就是参数Promise对象

    const p1 = Promise.resolve(123)
    console.log(p1)

    const p2 = Promise.resolve(new Promise((resolve, reject) => {
      resolve('success')
    }))
    
    console.log(p2)
    const p3 = Promise.resolve(new Promise((resolve, reject) => {
      reject('error')
    }))
    console.log(p3)
  </script>
</body>
</html>
```

## catch方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  
  <script>
    const p1 = new Promise((resolve, reject) => {
      reject('error')
    })
    // then方法中可以传递两个参数，也可以不传递，也可以只传递成功的回调函数
    // 也可以单独的用catch来专门指定失败的回调函数

    // catch方法也有返回值，和then方法的返回值类似
    // 1. 如果失败回调函数中没有返回值，则得到一个成功的Promise实例化对象，结果为undefined
    // 2. 如果失败回调函数中有非Promise的返回值，则得到一个成功的Promise实例化对象，结果为返回值数据
    // 3. 如果失败回调函数中有Promise的返回值，则得到一个Promise实例化对象，状态和值与失败回调的返回值一致
    const result1 = p1.catch(reason => {
      console.log(reason)
    })
  </script>
</body>
</html>
```

## finally方法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    // finally是ES9(ES2018)中新增的特性，表示无论Promise对象变成了fulfilled还是rejected状态，最终都会执行
    // finally方法的回调函数是不接受参数的

    const p1 = new Promise((resolve, reject) => {
      resolve('ok')
    })

    p1.then(value => {
      console.log(value)
    }).catch(reason => {
      console.log(reason)
    }).finally(() => {
      console.log('最终我被执行了...')
    })
  </script>
</body>
</html>
```

## 终止Promise链条

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    new Promise((resolve, reject) => {
      resolve(1)
    }).then(value => {
      console.log(value)
    }).then(value => {
      return new Promise(() => {}) // 通过返回一个pending状态的Promise终止Promise链
    }).then(value => {
      console.log(value)
    }).catch(reason => {
      console.log(reason)
    })
  </script>
</body>
</html>
```

## async和await

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>

    async function asyncTest() {
      // 情况1: 不写return，则返回一个成功类型的Promise，值为undefined

      // 情况2: return 非Promise对象的数据，则返回一个成功类型的Promise，值为返回的数据
      // return 100

      // 情况3: 返回的是Promise对象，则返回的Promise决定状态和数据
      // return new Promise((resolve, reject) => {
      //   resolve('ok')
      // })

      // 情况4: 抛出异常，则返回的Promise状态为rejected，值为抛出的异常
      // throw '出错啦'
    }

    let rs = asyncTest()
    console.log('asyncTest:', rs)

    // 1. async函数结合await表达式
    //   1.1 async函数中不一定要完全结合await
    //   1.2 有await的函数一定是async函数
    // 2. await相当于then，可以直接拿到成功Promise实例化对象的结果值
    // 3. await一定要写在async函数中，但是async函数中可以没有await
    // 4. 如果await表达式后面是Promise实例化对象，则await返回的是Promise的成功结果值
    // 5. 如果await表达式后面是非Promise实例化对象，则会直接将这个值作为await的返回值


    // 内部执行异步的功能，并且得到成功的结果数据值
    async function main() {
      // 1. 如果await右侧为一个非Promise类型的数据，则await后面的值是什么，得到的结果就是什么
      let rs1 = await 100
      console.log(rs1) // 100

      // 2. 如果await右侧为Promise成功类型的数据，则可以得到Promise的值
      let rs2 = await new Promise((resolve, reject) => {
        resolve('ok')
      } )
      console.log(rs2) // ok

      let rs3 = await Promise.resolve('okk')
      console.log(rs3) // okk

      // 3. 如果await右侧为Promise失败类型的数据，则需要通过try...catch来捕获
      try {
        let rs4 = await new Promise((resolve, reject) => {
          reject('error')
        })
        let rs5 = await Promise.reject('err')
      }catch(e) {
        console.log('catch:', e) // catch: error
      }

      console.log(1111) // 1111
    }

    main()
  </script>
</body>
</html>
```

## await表达式的执行顺序

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    async function main() {
      console.log(1111)
      let result = await Promise.resolve('ok')
      console.log(2222)
      console.log(result)
      console.log(3333)
      console.log(4444)
      let result1 = await Promise.resolve('okk')
      console.log(result1)
    }
    main()
    console.log(5555)
    console.log(6666)

    /**
     * 1111
     * 5555
     * 6666
     * 2222
     * ok
     * 3333
     * 4444
     * okk
     */
  </script>
</body>
</html>
```

## 宏队列和微队列

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    // JS中用来存储 待执行回调函数 的队列中 包含了两个特定的队列
    // 宏队列: 用来保存执行的宏任务(回调)，比如: 定时器、DOM事件操作、ajax
    // 微队列: 用来保存执行的微任务(回调)，比如: promise
    // JS执行的时候会区分两个队列
    // 1. 首先JS引擎会必须先将所有的同步代码都执行完
    // 2. 每次准备取出第一个宏任务执行之前，都需要将所有的微任务一个一个取出来执行
    // 3. 顺序为同-微-宏
  </script>
</body>
</html>
```

