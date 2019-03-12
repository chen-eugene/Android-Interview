#### 1、标准函数。
  - run：作用域中的接受者为this，可以省略，返回作用域中的最后一条代码的执行结果。
  ```
  stringVariable?.run{
    println("字符串的长度为$length")
  }
  ```

  - let：作用域的接受者为it，返回值为作用域中的最后一条代码的执行结果。
  ```
  stringVariable?.let{
    println("字符串长度为${it.length}")
  }
  ```
  
  - alse：作用域中的接受者为it，返回值为当前代用的对象，方便实现链式调用。
  ```
  "abc".also{
    println("The oringle String is $it") //"abc"
  }.also{
    println("The reverse String is ${it.reversed()}") //"cba"
  }.also{
    println("The length of the String is ${it.length}") //3
  }
  ```
  
  - apply：作用域中的接受者为this，返回值为当前调用的对象。
  ```
  val intent = Intent().apply{
    data = Uri.parse("")
  }
  ```
  
  - with：不是扩展函数，是正常函数，作用域中的接受者为this，返回值为作用域中的最后一条代码的执行结果。
  ```
  with(webview.setting){
    this?.javaScriptEnable = true
    this?.databaseEnable = true
  }
  ```

  - use：Closeable的扩展函数，用于IO操作，在use实现代码里面对代码块lambda代码块进行了try catch操作，并且在finally代码块中调用了close()方法，
   在进行IO操作的时候可以省去对于IOException的捕捉和释放。
  ```
  val stream = Files.newInputStream(Paths.get("/file.txt"))
  stream.buffered().reader().use{reader -> 
    println(reader.readText())
  }  
  ```

