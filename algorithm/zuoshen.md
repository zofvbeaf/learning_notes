## 栈和队列

### 仅用递归函数和栈操作逆序一个栈

```c++
class Solution
{
public:
    //递归函数一
    static int getAndRemoveStackLastElem(stack<int>& s)
    {
        int result = s.top();
        s.pop();
        if (s.empty())
            return result;
        else
        {
            int last = getAndRemoveStackLastElem(s);
            s.push(result);
            return last;
        }
    }

    //递归函数二
    static void reverseStack(stack<int>& s)
    {
        if (s.empty())
            return;

        int i = getAndRemoveStackLastElem(s);
        reverseStack(s);
        s.push(i);
    }
};
```

### **用栈求解汉诺塔问题**

```

```

