【NDC介绍】NDC是nested DiagnosticContext的缩写，意思是“嵌套的诊断上下文”。NDC是一种用来区分不同源代码中交替出现的日志的手段

【背景】当一个服务端程序同时记录好几个并行客户时，输出的日志会混杂在一起难以区分。但如果不同上下文的日志入口拥有一个特定的标识，则可以解决这个问题。NDC就是在这种情况下发挥作用。注意NDC是以线程为基础的，每个线程拥有一个NDC，每个NDC的操作仅对执行该操作的线程有效。

【使用】NDC的几个有用的方法是：push、pop、get和clear。注意它们都是静态函数

1. Push可以让当前线程进入一个NDC，如果该NDC不存在，则根据push的参数创建一个NDC并进入;如果再调用一次push，则进入子NDC;
2. Pop可以让当前线程从上一级NDC中退出，但是一次只能退出一级。
3. Clear可以让当前线程从所有嵌套的NDC中退出。
4. Get可以得到当前NDC的名字，如果有嵌套，则不同级别之间的名字用空格隔开

【例子】在记录日志的时候，可以从NDC中得知当前线程的嵌套关系。
```cpp
std::cout<< "1.empty NDC: " <<NDC::get()<< std::endl;
NDC::push("context1");

std::cout<< "2.push context1: " <<NDC::get()<< std::endl;
NDC::push("context2");

std::cout<< "3.push context2: " <<NDC::get()<< std::endl;
NDC::push("context3");

std::cout<< "4.push context3: " <<NDC::get()<< std::endl;
std::cout<< "5.get depth: " <<NDC::getDepth() <<std::endl;
std::cout<< "6.pop: " << NDC::pop()<< std::endl;
std::cout<< "7.after pop:"<<NDC::get()<<std::endl;

NDC::clear();
std::cout<< "8.clear: " << NDC::get() <<std::endl;
```