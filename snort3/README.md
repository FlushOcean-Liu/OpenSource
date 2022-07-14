# snort3详解

## 主线流程

线程启动函数main.cc:main_loop  
![](./img/main_loop.jpg)

  
线程函数句柄main.cc:handle
![](./img/main_thread_handle.jpg)


线程start函数main.cc:Pig::start()
![](./img/thread_start.jpg)

  
线程函数analyzer.cc:Analyzer::operator()
![](./img/thread_function.jpg)



