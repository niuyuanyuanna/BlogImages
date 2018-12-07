---
title: 数据结构之栈和队列
date: 2018-04-06 13:55:22
tags:
- 数据结构
- C++
categories: Data Structure
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/data_structure.jpg
---

# 栈（stack）

## 基础概念

栈仍然是一个序列，遵循后进先出的原则。可以基于向量或列表派生，共有三种操作方法：

```
push()    入栈   insert(size(), e)
pop()     出栈   remove(size() -1)       
top()     取顶操作    (*this)[size() -1]
```

## 栈的应用

- 逆序输出：conversion 输出次序与处理过程颠倒，递归深度和输出长度不易预知
- 递归嵌套：stack permutation + parenthesis 具有自相似性的问题可递归描述，但分支位置和嵌套深度不固定
- 延迟缓冲：evaluation 线性扫描算法模式中，在预读足够长之后，才能确定可处理的前缀

### 逆序输出

逆序输出的一个应用为进制转换，将整除后的余数入栈，最后出栈。
```c++
void convert(Stack<char> & S, _int64){
    static char digit[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};
    while(n>0){
        S.push(digit[n % base]);
        n /= base;
    }
}
main(){
    Stack<char> S;
    convert(S, n, base);
    while(!S.empty())
        print("%c", S.pop());    //逆序逆序输出
}
```

### 递归嵌套
括号匹配是递归嵌套的一种应用。找到括号的方法即是遇到“（”则入栈，遇到“）”则出栈。
```c++
bool paren(const char exp[], int lo, int hi){
    Stack<char> S;
    for(int i = lo; i < hi; i++){
        if('(' == exp[i])          //若遇到(则进栈
            S.push(exp[i]);
        else if(!S.empty())        //若遇到)且栈非空则出栈
            S.pop();
        else return false;         //若遇到)时栈已空，则不匹配
    }
    return S.empty();              //只有栈为空时匹配
}
```
这种算法可以便捷的推广到多种括号并存的情况，也可以引申至HTML语言中的标签上。

#### 栈混洗
将A栈的栈顶弹出存入S中转栈，然后将S栈的栈顶弹出存入到B栈，完成栈的混洗。若A全部转入S后弹出，A和B的栈顶和栈底顺序正好颠倒，若S弹出的开始时间是随机的，那么B栈中存储的数据顺序就是随机的。因此栈混洗的情况有catalan数种：$\frac{2n!}{\left ( n + 1 \right )!\times n!}$。
在级数排序时，有几种排序是禁形，如A中元素为123，B中为312，此时B不可能通过混洗得到。若一个排列是栈混洗，当且仅当其排序不包含312的形式。
判定是否为栈混洗的时候：

- 每次S.pop()之前，判断S是否为空
- 若需弹出的元素在S中，却非顶元素

### 延迟缓冲
中缀表达式求值为延迟缓冲的应用
以算数表达式为例 ：求值算法=栈+线性扫描
实现算法：用两个栈分别存储运算数和运算符
```c++
float evaluate(char* S, char* & RPN){        //RPN转换
    Stack<float> opnd;                       //运算数栈
    Stack<char> optr;                        //运算符栈
    optr.push('\0');                          //尾哨兵
    while(!optr.empty()){
        if(isdigit(*S))
            readNumber(S, opnd);             //读入操作数，可能为多位
            append(RPN, opend.top());        //接入RPN
        else{
            switch(orderBetween(optr.top(), *S)){ //分别处理，判断当前运算符和栈顶运算符之间的优先级高低       
            }
        }
    }
    return opnd.pop();
}
```
在读入操作数时，需要进行处理，因为读入数可能是多位。如果当前number为一位数字，就入栈；如果是多位数字就需要对这几位数字进行处理：
```c++
float evaluate(char* S, char* & RPN){        //RPN转换
    Stack<float> opnd;                       //运算数栈
    Stack<char> optr;                        //运算符栈
    optr.push('\0');                         //尾哨兵
    int numBits = 0;                         //用于存储数字的位数
    while(!optr.empty()){
        if(isdigit(*S))
            numBits += 1;
            readNumber(S, opnd, numBits);    //读入操作数，可能为多位
            append(RPN, opend.top());        //接入RPN
        else{
            numBits = 0;
            switch(orderBetween(optr.top(), *S)){ //分别处理，判断当前运算符和栈顶运算符之间的优先级高低       
            }
        }
    }
    return opnd.pop();
}
float readNumber(char* S, Stack opnd, int num){  
    if(num < 2){
        opnd.push(*S);
    }else{
        float oOpnd = S;
        for(i=2; i<=n; i++){
            float bitNum = opnd.pop()*10^(i-1)
            oOpnd += bitNum;
        }
        opnd.push(oOpnd);        
    }        
}
```
判断运算符优先级采用表格存储，行为当前运算符，列为栈顶运算符 。

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/dataStructure/20181203.png" width=75%/>
</center>

对于不同优先级的处理方法：

```c++
switch(orderBetween(optr.top(), *S)){
    case '<':                                  //栈顶优先级低，推迟计算，当前运算符入栈
        optr.push(*S);
        S++;
        break;
    case '=':                                  //优先级相等，脱括号，接收下一字符。只有在括号和\0的时候才有=出现
        optr.pop();
        S++;
        break;
    case '>':                                  //栈顶优先级高，栈顶运算符出栈，执行计算
        char op = optr.pop();
        append(RPN, op);     //接入RPN
        if('!' == op) 
            opnd.push(calcu(op, opnd.pop()))    //一元运算符入栈
        else{
            float pOpnd2 = opnd.pop();
            float pOpnd1 = opnd.pop();
            opnd.push(calcu(pOpnd1, op, pOpnd2));
        }
        break;
}
```

逆波兰表达式（RPN）：不使用括号表示优先级，所有操作数的次序和在中缀表达式中的次序相同。

# 队列

## 队列基础

队列接口与实现，先进先出，后进后出，其操作为：
```
dequeue()                                 头部删除
front()                                   头部查询
enqueue(e)                                尾部插入 
rear()                                    尾部查询 
```

其模板类可以直接基于向量或列表派生。
```c++
template <typename T> class Queue:public List<T>{
    public:   //size()和empty()可以直接继承
    T dequeue(){
        return remove(first());
    }
    T & front(){
        return first()->data;
    }
}
```
