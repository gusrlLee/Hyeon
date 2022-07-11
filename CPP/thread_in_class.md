# 클래스 안에 thread running 하는 법 (C++)  
₩₩₩  
#include <thread>  
void Test::run()  
{  
    std::thread t1(&Test::t1, this, arg1, arg2);  
    std::thread t2(&Test::t2, this, arg1, arg2);  
    ...  
}  
₩₩₩  
이런 식으로 thread를 지정 해서 넣어 주어야 한다.  
