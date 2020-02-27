---
layout: post
categories: 数据结构与算法
title: Stack 经典面试题之判断字符串是否合法
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---

相信大家都知道，Stack (栈): 后进先出( Last In First Out )，也就是说后面进来的，会先出去。
<!--more-->
每次说到栈，贪吃的阿粉就会想起烙饼这件事。每次阿粉的母亲烙饼的时候，先烙好的饼会放在最下面，后面烙好的饼会放在上面，还在烙饼的时候，我就想吃所以被我吃到的就是最上面的饼。

感觉这个过程是不是和栈这种数据结构很像~

对于 Stack 来说，经典的面试题莫过于，判断字符串是否合法了。

判断字符串是否合法是这样的：有一个字符串，它只包含大中小括号，那么符号 ([)] 这样是不合法的，合法的应该是这样: ([]) ，同样 ([]){ 这样的符号也是不合法的
基于以上的共识，咱们先考虑使用数组的方式，来分析一下。

1. 定义一个初始值，如果刚开始输入的就是 ( 或者 { 或者 [ ，那么我们不能立刻判断到它就是不合法的，因为它需要等待匹配，如果到最后还是没有匹配上，那就是不合法的;如果刚开始输入的是 ) 或者 } 或者 ] ，我们立刻就能知道这是不合法的。

2. 如果此时输入了 ( 和 [ ，初始值应该 ++ ，接下来输入的是右边的符号的话应该是 ] 而不是 ) ，此时需要进行判断第三个输入的字符是否匹配第二个，只有第二个也匹配之后才需要进行匹配第一个字符。

3.  如果匹配成功，则初始值应该 - - ，所有字符串匹配完毕之后，需要看初始值是否为最初赋予的值，如果是则说明所有符号都是合法的，否则说明还有符号没有匹配上，则不合法
经过这样的分析之后，写代码应该就比较好写了，比如我们可以这样实现:

```java
/**
 * 判断字符串是否合法
 *      比如: "([)]" 不合法, "[()]" 合法
 */
public class IsValidString {
    public static void main(String[] args) {
        // 定义字符串的内容
        String symbol="([]){";
        // 调用判断方法
        boolean result=isValid(symbol);
        System.out.print(result);
    }
    public static  boolean isValid(String s){
        // 用来接收传入的值
        char[] arr = s.toCharArray();
        // 定义一个数组,用来存放传入的字符串,长度为传入的字符串的值
        char[] stack = new char[s.length()];
        // 定义 stackEnd 为 -1 是为了让第一个元素能够进入数组,即 stackEnd++ 值为 0
        int stackEnd=-1,length=s.length();
        for(int i=0;i<length;i++){
            // 如果刚开始是左括号,左中括号等符号,则不能直接判断为该符号不合法,而是放入数组,等待匹配
            if(arr[i]=='(' || arr[i]=='[' || arr[i]=='{'){
                stackEnd++;
                stack[stackEnd]=arr[i];
            }
            // 如果刚开始就是右括号,右中括号等符号,则不合法,直接返回 false
            else if((arr[i]==']' || arr[i]==')' || arr[i]=='}') && stackEnd==-1){
                return false;
            }
            else {
                // 分情况来进行匹配
                if(arr[i]==')' && stack[stackEnd]=='('){
                    stackEnd--;
                }
                else if(arr[i]==']' && stack[stackEnd]=='['){
                    stackEnd--;
                }
                else if(arr[i]=='}' && stack[stackEnd]=='{'){
                    stackEnd--;
                }
                else{
                    // 如果都匹配不到,说明该符号不合法,则直接返回 false
                    return false;
                }
            }
        }
        if(stackEnd!=-1){
            // 如果最后结果不等于 -1 ,说明 stackEnd 中还有符号没有被匹配到,则也是不合法
            return false;
        }
        else
            return true;
    }
}
```

可以明显看到，使用数组的话，会有很多 if ， else ， else if ，阿粉怎么闻到了坏代码的味道。

有没有更巧妙的方法呢？看下面：

```java
/**
 * 判断字符串是否合法
 *      比如: "([)]" 不合法, "[()]" 合法
 */
public class IsValidString {
    public static void main(String[] args) {
        // 定义字符串的内容
        String symbol="([]){";
        // 调用判断方法
        boolean result=isValid(symbol);
        System.out.print(result);
    }
    public static boolean isValid(String s){
        // 定义一个空栈
        Stack<Character> stack=new Stack<>();
        // 定义 map ,用来存放匹配的符号
        Map<Character,Character> map=new HashMap<>();
        char[] arr=s.toCharArray();
        map.put(')','(');
        map.put(']','[');
        map.put('}','{');
        for (int i=0;i<arr.length;i++){
            // 如果 map 中不包含进入的符号,说明是左边的符号,直接入栈即可
            if (!map.containsKey(arr[i])){
                stack.push(arr[i]);
            }else {
                // 如果进入的符号和栈顶的元素不匹配,则说明符号不合法
                if (map.get(arr[i])!=stack.pop()){
                    return false;
                }
            }
        }
        // 最后判断栈是否为空,如果为空,说明所有符号都已匹配完毕,全都合法
        // 如果栈不为空,说明还有符号没有匹配到,则不合法
        if (stack.empty()){
            return true;
        }else {
            return false;
        }

    }
}
```

阿粉发现使用栈来实现，代码简单易读了好多~

判断字符串是否合法，阿粉已经掌握了，你呢？

或者你有没有更好的实现方法，欢迎交流