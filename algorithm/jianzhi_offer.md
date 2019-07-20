## 剑指offer

> 按牛客网的顺序。

### 二维数组中的查找

> 在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

```c++
class Solution {
public:
  bool Find(int target, vector<vector<int> > array) {
    if(array.empty() || array[0].empty() || array[0][0] > target) 
      return false;
    int n = array.size(), m = array[0].size();
    int i = 0, j = m-1;
    while(i < n && j >= 0) {
      if(array[i][j] == target) 
        return true;
      else if(array[i][j] < target) 
        ++i;
      else 
        --j;
    }
    return false; 
  }
};
```

### 替换空格

> 请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

```c++
class Solution {
public:
        void replaceSpace(char *str,int length) {
                if(!str) return;
          int old_len = 0;
          int space_num = 0;
          while(str[old_len] != '\0') {
            if(str[old_len++] == ' ') 
              ++space_num;
          }
          int new_len = space_num*2 + old_len;
          if(new_len > length) 
            return;
          
          for(int i = old_len, j = new_len; i >= 0; --i, --j) {
            if(str[i] == ' ') {
              str[j] = '0';
              str[j-1] = '2';
              str[j-2] = '%';
              j -= 2;
            }
      else 
        str[j] = str[i];
          }
        }
};
```

### 从尾到头打印链表

> 输入一个链表，按链表值从尾到头的顺序返回一个ArrayList。

```c++
/**
*  struct ListNode {
*        int val;
*        struct ListNode *next;
*        ListNode(int x) :
*              val(x), next(NULL) {
*        }
*  };
*/
class Solution {
public:
    void addValue(vector<int>& ans, ListNode* node) {
        if(!node) return;
        addValue(ans, node->next);
        ans.push_back(node->val);
    }
    
    vector<int> printListFromTailToHead(ListNode* head) {
        vector<int> ans;
        if(!head) return ans;
        addValue(ans, head);
        return ans;
    }
};
```

### 重建二叉树

> 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列`{1,2,4,7,3,5,6,8}`和中序遍历序列`{4,7,2,1,5,3,8,6}`，则重建二叉树并返回。

```c++
/**
 * Definition for binary tree
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
  TreeNode* solve(vector<int>& pre, vector<int>& vin,
                  int pstart, int pend, int vstart, int vend) {
    if(pstart > pend || vstart > vend)
      return nullptr;
    TreeNode* root = new TreeNode(pre[pstart]); 
    int vroot_pos = vstart;
    while(vroot_pos <= vend) {
      if(vin[vroot_pos] == pre[pstart]) 
        break;
      else 
        vroot_pos++;
    }
    root->left = solve(pre, vin, pstart+1, pstart+vroot_pos-vstart, vstart, vroot_pos-1);
    root->right = solve(pre, vin, pend-(vend-vroot_pos)+1, pend, vroot_pos+1, vend); 
    return root;
  }

        TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> vin) {
                if(pre.empty() || vin.empty() || pre.size() != vin.size())
                        return nullptr;
    return solve(pre, vin, 0, pre.size()-1, 0, vin.size()-1);
        }
};
```

### 用两个栈实现队列

> 用两个栈来实现一个队列，完成队列的`Push`和`Pop`操作。 队列中的元素为`int`类型。

```c++
class Solution
{
    public:
    void push(int node) {
        stack1.push(node);
    }

    int pop() {
        if(stack2.empty()) {
            while(!stack1.empty()) {
                int temp = stack1.top();
                stack2.push(temp);
                stack1.pop();
            }
        }
        int res = stack2.top();
        stack2.pop();
        return res;
    }

    private:
    stack<int> stack1;
    stack<int> stack2;
};
```

### 旋转数组的最小数字

> 把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组`{3,4,5,1,2}`为`{1,2,3,4,5}`的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

```c++
class Solution {
    public:
    int minNumberInRotateArray(vector<int> rotateArray) {
        int left = 0, right = rotateArray.size() - 1, mid = 0; 
        while(rotateArray[left] >= rotateArray[right]) {
            if(right - left == 1) 
                return rotateArray[right];
            if(rotateArray[left] == rotateArray[mid]
               && rotateArray[right] == rotateArray[mid]) {
                int res = rotateArray[left];
                while(left <= right) {
                    res = res < rotateArray[left] ? res : rotateArray[left];
                    ++left;
                }
                return res;
            }
            int mid = (left+right) >> 1;
            if(rotateArray[left] <= rotateArray[mid]) 
                left = mid;
            else if(rotateArray[mid] <= rotateArray[right])
                right = mid;
        }
        return rotateArray[mid];
    }
};
```

### 斐波那契数列

> 大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。n<=39

```c++
class Solution {
    public:
    int Fibonacci(int n) {
        if(n < 2) return n;
        int x = 0, y = 1;
        while(--n) {
            int temp = x;
            x = y;
            y += temp;
        }
        return y;
    }
};
```

### 跳台阶

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

```c++
class Solution {
public:
    int jumpFloor(int number) {
        int n = number;
        if(n < 2) return n;
        int x = 1, y = 1;
        while(--n) {
          int temp = x;
          x = y;
          y += temp;
        }
        return y;
    }
};
```

### 变态跳台阶

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

```c++
class Solution {
public:
    int jumpFloorII(int number) {
        return 1 << (number-1);
    }
};
```

### 矩形覆盖

> 我们可以用`2*1`的小矩形横着或者竖着去覆盖更大的矩形。请问用`n`个`2*1`的小矩形无重叠地覆盖一个`2*n`的大矩形，总共有多少种方法？

```c++
class Solution {
    public:
    int rectCover(int number) {
        if(number <= 0) 
            return 0;
        if(number <= 3) 
            return number;
        int x = 1, y = 2;
        number -= 2;
        while(number--) {
            int temp = y;
            y += x;
            x = temp;
        }
        return y;
    }
};
```

### 二进制中1的个数

> 输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

```c++
class Solution {
public:
     int NumberOf1(int n) {
         int ans = 0;
         while(n) {
             ++ans;
             n = (n-1) & n;
         }
         return ans;
     }
};
```

### 数值的整数次方

> 给定一个`double`类型的浮点数`base`和`int`类型的整数`exponent`。求`base`的`exponent`次方。

```c++
class Solution {
public:
    double Power(double base, int exponent) {
        if(base == 0) {
            if(exponent == 0) return 1;
            else return 0;
        }
        if(exponent == 0) {
            return 1;
        }
        else if(exponent < 0) {
            exponent = -exponent;
            base = 1.0/base;
        }
        double ans = 1;
        while(exponent) {
            if(exponent & 1) {
                ans = ans * base;
            }
            base *= base;
            exponent >>= 1;
        }
        return ans;
    }
};
```

### 调整数组使奇数位于偶数前面

> 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

```c++
class Solution {
public:
    void reOrderArray(vector<int> &array) {
        vector<int> temp_evens;
        for(auto it = array.begin(); it != array.end();) {
            if((*it)%2 == 0) {
                temp_evens.push_back((*it));
                array.erase(it);
            }
            else {
                ++it;
            }
        }
        for(auto& i : temp_evens) {
            array.push_back(i);
        }
    } 
};
```

+ 若不用保证奇数和奇数，偶数和偶数之间的相对位置不变。

```c++
class Solution {
public:
    void reOrderArray(vector<int> &array) {
         int l = 0, r = array.size() - 1;
         while (l < r) {
             while (l < r && array[l] % 2 == 1) ++l;
             while (l < r && array[r] % 2 == 0) --r;
             if (l < r) swap(array[l], array[r]);
         }
    }
};
```

### 链表中倒数第k个节点

> 输入一个链表，输出该链表中倒数第k个结点。

```c++
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* FindKthToTail(ListNode* pListHead, int k) {
        if(!pListHead || k == 0) 
            return nullptr;
        ListNode* pa = pListHead;
        ListNode* pb = nullptr;
        for(int i = 0; i < k-1; ++i) {
            pa = pa->next;
            if(!pa) {
                return nullptr;
            }
        }
        pb = pListHead;
        while(pa->next != nullptr) {
            pa = pa->next;
            pb = pb->next;
        }
        return pb;
    }
};
```

### 反转链表

> 输入一个链表，反转链表后，输出新链表的表头。

```c++
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* ReverseList(ListNode* head) {
        if(!head || head->next == nullptr) {
            return head;
        }
        ListNode* temp_head = head;
        ListNode* ans = nullptr;
        ListNode* temp_prev = nullptr;
        while(temp_head) {
            ListNode* temp_next = temp_head->next;
            if(!temp_next) 
                ans = temp_head;
            temp_head->next = temp_prev;
            temp_prev = temp_head;
            temp_head = temp_next;
        }
        return ans;
    }
};
```

### 合并两个排序的链表

> 输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

```c++
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        ListNode *dummy = new ListNode(0);
        ListNode *cur = dummy;
        while (pHead1 != NULL && pHead2 != NULL) {
            if (pHead1 -> val < pHead2 -> val) {
                cur -> next = pHead1;
                pHead1 = pHead1 -> next;
            }
            else {
                cur -> next = pHead2;
                pHead2 = pHead2 -> next;
            }
            cur = cur -> next;
        }
        cur -> next = (pHead1 ? pHead1 : pHead2);
        cur = dummy -> next;
        delete dummy;
        return cur;
    }
};
```

### 树的子结构

> 输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

```c++
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    bool HasSubtree(TreeNode* pRoot1, TreeNode* pRoot2)
    {
        if(!pRoot2 || !pRoot1) {
            return false;
        }
        
        if(pRoot1->val == pRoot2->val) {
            bool left = true, right = true;
            if(pRoot2->left) {
                left = HasSubtree(pRoot1->left, pRoot2->left);
            }
            if(pRoot2->right) {
                right = HasSubtree(pRoot1->right, pRoot2->right);
            }
            if(left && right)
                return true;
        }
        return HasSubtree(pRoot1->left, pRoot2) || HasSubtree(pRoot1->right, pRoot2);
    }
};
```

### 二叉树的镜像

> 操作给定的二叉树，将其变换为源二叉树的镜像。

```c++
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    void Mirror(TreeNode *pRoot) {
        if(!pRoot) {
            return;
        }
        TreeNode* temp = pRoot->left;
        pRoot->left = pRoot->right;
        pRoot->right = temp;
        Mirror(pRoot->left);
        Mirror(pRoot->right);
    }
};
```

### 顺时针打印矩阵

> 输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

```c++
class Solution {
public:
    void printMatrixCircle(vector<vector<int> >& matrix, int start, vector<int>& ans) {
        int rows = matrix.size(), columns = matrix[0].size();
        int endr = rows-start-1;
        int endc = columns-start-1;
        
        for(int i = start; i <= endc; ++i) {
            ans.push_back(matrix[start][i]);
        }
        
        for(int i = start+1; i <= endr; ++i) {
            ans.push_back(matrix[i][endc]);
        }
     
        if(start < endr) {
            for(int i = endc-1; i >= start; --i) {
                ans.push_back(matrix[endr][i]);
            }    
        }
        
        if(start < endc) {
            for(int i = endr-1; i > start; --i) {
                ans.push_back(matrix[i][start]);
            }
        }
    }
    
    vector<int> printMatrix(vector<vector<int> >& matrix) {
        vector<int> ans;
        if(matrix.empty()) {
            return ans;
        }
        if(matrix.size() == 1) {
            ans = matrix[0];
            return ans;
        }
        int rows = matrix.size(), columns = matrix[0].size();
        int start = 0;
        while(start*2 < rows && start*2 < columns) {
            printMatrixCircle(matrix, start, ans);
            ++start;
        }
        return ans;
    }
};
```

### 包含min函数的栈

> 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

```c++
class Solution {
public:
    void push(int value) {
        data.push(value);
        if(min_data.empty() || min_data.top() > value) {
            min_data.push(value);
        }
        else {
            min_data.push(min_data.top());
        }
    }
    
    void pop() {
        data.pop();
        min_data.pop();
    }
    int top() {
        return data.top();
    }
    int min() {
        return min_data.top();
    }
 private:
    stack<int> data;
    stack<int> min_data;
};
```

### 栈的压入弹出序列

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

```c++
class Solution {
public:
    bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        if(pushV.size() != popV.size()) {
            return false;
        }
        if(pushV.empty()) {
            return true;
        }
        stack<int> temp_stack;
        int i = 0, j = 0;
        while(i < pushV.size()) {
            temp_stack.push(pushV[i]);
            while(j < popV.size() && temp_stack.top() == popV[j]) {
                temp_stack.pop();
                ++j;
            }
            ++i;
        }
        return temp_stack.empty();
    }
};
```

### 从上往下打印二叉树

> 从上往下打印出二叉树的每个节点，同层节点从左至右打印。

```c++
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    vector<int> PrintFromTopToBottom(TreeNode* root) {
        vector<int> ans;
        if(!root) {
            return ans;
        }
        queue<TreeNode*> temp_queue;
        temp_queue.push(root);
        while(!temp_queue.empty()) {
            TreeNode* temp_node = temp_queue.front();
            ans.push_back(temp_node->val);
            temp_queue.pop();
            if(temp_node->left) {
                temp_queue.push(temp_node->left);
            }
            if(temp_node->right) {
                temp_queue.push(temp_node->right);
            }
        }
        return ans;
    }
};
```

### 二叉搜索树的后序遍历序列

> 输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。

```c++
class Solution {
public:
    bool VerifySquenceOfBSTCore(vector<int>& sequence, int x, int y) {
        if(x >= y) 
            return true;
        int root = sequence[y];
        int left = 0;
        for(; left < y; ++left) {
            if(sequence[left] > root) {
                break;
            }
        }
        int right = left;
        --left;
        for(; right < y; ++right) {
            if(sequence[right] < root) {
                return false;
            }
        }
        return VerifySquenceOfBSTCore(sequence, x, left) 
               && VerifySquenceOfBSTCore(sequence, right, y-1);
    }
    
    bool VerifySquenceOfBST(vector<int> sequence) {
        if(sequence.empty()) {
            return false;
        }
        return VerifySquenceOfBSTCore(sequence, 0, sequence.size()-1); 
    }
};
```

### 二叉树中和为某一值的路径

> 输入一颗二叉树的跟节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)

```c++
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    void FindPathCore(vector<int>& path, vector<vector<int> >& ans, int tempsum,
                      TreeNode* root,int expectNumber) {
        if(!root)
            return;
        tempsum += root->val;
        path.push_back(root->val);
        if(root->left == nullptr && root->right == nullptr && tempsum == expectNumber) {
            ans.push_back(path);
        }
        FindPathCore(path, ans, tempsum, root->left, expectNumber);
        FindPathCore(path, ans, tempsum, root->right, expectNumber);
        path.pop_back();
    }
    
    vector<vector<int> > FindPath(TreeNode* root,int expectNumber) {
        vector<vector<int> > ans;
        vector<int> path;
        if(!root)
            return ans;
        FindPathCore(path, ans, 0, root, expectNumber);
        return ans;
    }
};
```

### 复杂链表的复制

> 输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

```c++
/*
struct RandomListNode {
    int label;
    struct RandomListNode *next, *random;
    RandomListNode(int x) :
            label(x), next(NULL), random(NULL) {
    }
};
*/
class Solution {
public:
    RandomListNode* Clone(RandomListNode* pHead)
    {
        if(!pHead) {
            return pHead;
        }

        RandomListNode* temp_head = pHead;
        while(temp_head) {
            RandomListNode* clone_node = new RandomListNode(temp_head->label);
            clone_node->next = temp_head->next;
            temp_head->next = clone_node;
            temp_head = clone_node->next;
        }

        temp_head = pHead;
        while(temp_head) {
            RandomListNode* clone_node = temp_head->next;
            if(temp_head->random) {
                clone_node->random = temp_head->random->next;
            }
            temp_head = clone_node->next;
        }

        temp_head = pHead;
        RandomListNode* ans = pHead->next;
        while(temp_head) {
            RandomListNode* clone_node = temp_head->next;
            temp_head->next = clone_node->next;
            temp_head = temp_head->next;
            if(temp_head) {
                clone_node->next = temp_head->next;
            }
        }
        
        return ans;
    }
};
```

### 二叉搜索树与双向链表

> 输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

```c++
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    void ConvertCore(TreeNode* pRoot, TreeNode** pLastNode) {
        if(!pRoot)
            return;
        if(pRoot->left) {
            ConvertCore(pRoot->left, pLastNode);
        }
        pRoot->left = *pLastNode;
        if(*pLastNode) {
            (*pLastNode)->right = pRoot;
        }
        *pLastNode = pRoot;
        if(pRoot->right) {
            ConvertCore(pRoot->right, pLastNode);
        }
    }
    
    TreeNode* Convert(TreeNode* pRootOfTree)
    {
        TreeNode* pLastNode = nullptr;
        ConvertCore(pRootOfTree, &pLastNode);
        while(pLastNode && pLastNode->left) {
            pLastNode = pLastNode->left;
        }
        return pLastNode;
    }
};
```

### 字符串的排列

> 输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

```c++
class Solution {
public:
    void PermutationCore(string str, vector<string>& ans, int start) {
        if(start >= str.size()) {
            ans.push_back(str);
            return;
        }
        unordered_set<char> temp_chars;
        sort(str.begin() + start, str.end());
        for(int i = start; i < str.size(); ++i) {
            if(temp_chars.count(str[i])) {
                continue;
            }
            temp_chars.insert(str[i]);
            char c = str[start];
            str[start] = str[i];
            str[i] = c;
            PermutationCore(str, ans, start+1);
            c = str[start];
            str[start] = str[i];
            str[i] = c;
        }
    }
    
    vector<string> Permutation(string str) {
        vector<string> ans;
        if(str.empty()) {
            return ans;
        }
        PermutationCore(str, ans, 0);
        return ans;
    }
};
```

### 数组中出现超过一半的数字

> 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

```c++
class Solution {
public:
    int MoreThanHalfNum_Solution(vector<int> numbers) {
        if(numbers.empty()) {
            return 0;
        }
        int x = numbers[0], y = 1;
        
        for(int i = 1; i < numbers.size(); ++i) {
            if(numbers[i] != x) {
                --y;
                if(y <= 0) {
                    x = numbers[i];
                    y = 1;
                }
            }
            else {
                ++y;
            }
        }
        y = 0;
        for(int i = 0; i < numbers.size(); ++i) {
            if(numbers[i] == x)
                ++y;
        }
        return y > numbers.size()/2 ? x : 0;
    }
};
```

### 最小的k个数

> 输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

```c++
class Solution {
public:
    int partition(vector<int>& input, int left, int right) {
        int pivot = input[left];
        while(left < right) {
            while(left < right && input[right] >= pivot) --right;
            input[left] = input[right];
            while(left < right && input[left] <= pivot) ++left;
            input[right] = input[left];
        }
        input[left] = pivot;
        return left;
    }
    
    vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
        if(input.size() < k || k <= 0) {
            return vector<int>();
        }
        else if(input.size() == k) {
            return input;
        }
        int start = 0, end = input.size()-1;
        int index = partition(input, start, end);
        while(index != k-1) {
            if(index > k-1) {
                end = index-1;
            }
            else {
                start = index+1;
            }
            index = partition(input, start, end);
        }
        return vector<int>(input.begin(), input.begin()+k);
    }
};
```

+ 若不可以改变原数组

```c++
class Solution {
public:
    vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
        vector<int> ans;
        if(input.size() < k || k <= 0) {
            return ans;
        }
        else if(input.size() == k) {
            return input;
        }
        priority_queue<int> que;
        for(auto& i : input) {
            if(que.size() < k) {
                que.push(i);
            } else if(!que.empty() && i < que.top()) {
                que.pop();
                que.push(i);
            }
        }
         
        while(!que.empty()) {
            ans.push_back(que.top());
            que.pop();
        }
        return ans;
    }
};
```

### 连续子数组的最大和

> HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)

```c++
class Solution {
public:
    int FindGreatestSumOfSubArray(vector<int> array) {
        if(array.empty()) {
            return INT_MIN;
        }
        int ans = array[0], count = array[0];
        for(int i = 1; i < array.size(); ++i) {
            count += array[i];
            if(count > ans) {
                ans = count;
            }
            if(count < 0) {
                count = 0;
            }
        }
        return ans;
    }
};
```

### 从1到n整数中1出现的次数

> 求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。

```c++
class Solution {
public:
    int NumberOf1Between1AndN_Solution(int n)
    {
        if(n < 0) {
            n = 0-n;
        }
        if(n < 10) {
            return n >= 1;
        }   
        int ans = 0, base = 1, m = n;
        while(m) {
            int x = m%10;
            m /= 10;
            ans += m*base;
            if(x == 1) ans += (n%base)+1;
            else if(x > 1) ans += base;
            base *= 10;
        }
        return ans;
    }
};
```

### 把数组排成最小的数

> 输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。

```c++
class Solution {
public:
    string PrintMinNumber(vector<int> numbers) {
        sort(numbers.begin(), numbers.end(), [](const int& lhs, const int& rhs){
            string left = std::to_string(lhs);
            string right = std::to_string(rhs);
            return (left+right) < (right+left);
        });
        string ans;
        for(auto& i : numbers)
            ans += std::to_string(i);
        return ans;
    }
};
```

### 丑数

> 把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。

```c++
class Solution {
public:
    int GetUglyNumber_Solution(int index) {
        if(index <= 0) 
            return 0;
        vector<int> temp {0, 1}; // c++11
        int p2 = 1, p3 = 1, p5 = 1;
        while(temp.size() <= index) {
            while(2*temp[p2] <= temp.back()) ++p2;
            while(3*temp[p3] <= temp.back()) ++p3;
            while(5*temp[p5] <= temp.back()) ++p5;
            int x = std::min(std::min(2*temp[p2], 3*temp[p3]), 5*temp[p5]);
            temp.push_back(x);
        }
        return temp[index];
    }    
};
```

### 第一次只出现一次的字符的位置

> 在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1（需要区分大小写）.

```c++
class Solution {
public:
    int FirstNotRepeatingChar(string str) {
        unordered_map<char, int> flag;
        for(int i = 0; i < str.size(); ++i) {
            if(!flag.count(str[i])) {
                flag[str[i]] = i;
            }
            else if(flag[str[i]] >= 0) {
                flag[str[i]] = -1;
            }
        }
        for(int i = 0; i < str.size(); ++i) {
            if(flag[str[i]] >= 0)
                return flag[str[i]];
        }
        return -1;
    }
};
```

### 数组中的逆序对

> 在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007

```c++
class Solution {
public:
    int InversePairs(vector<int> data) {
        int ans = 0;
        vector<int> dup(data);
        return merge(dup, data, 0, data.size()-1);
    }
    int merge(vector<int>& dup, vector<int>& data, int left, int right) {
        if(left >= right) {
            dup[left] = data[left];
            return 0;
        }
        int mid = (left+right)/2;
        int left_num = merge(dup, data, left, mid);
        int right_num = merge(dup, data, mid+1, right);
        int i = mid, j = right, k = right, cnt = (left_num+right_num)%1000000007; 
        while(i >= left && j > mid) { // important : from right to left
            if(data[i] <= data[j]) {
                dup[k--] = data[j--];
            }
            else {
                dup[k--] = data[i--];
                // important: 表示右边的子数组比data[i]小的数的个数
                // 如果从左往右统计则可以改为 (cnt + mid - i + 1) 表示当前左边大于data[j]的元素个数
                cnt = (cnt + j - mid)%1000000007;  
            }
        }
        while(i >= left) {
            dup[k--] = data[i--];
        }
        while(j > mid) {
            dup[k--] = data[j--];
        }
        for(i = left; i <= right; ++i)
            data[i] = dup[i];
        return cnt;
    }
};
```

### 两个链表的第一个公共节点

> 输入两个链表，找出它们的第一个公共结点。

```c++
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* FindFirstCommonNode(ListNode* pHead1, ListNode* pHead2) {
        if(!pHead1 || !pHead2) {
            return nullptr;
        }
        int x = 0, y = 0;
        ListNode *p1 = pHead1, *p2 = pHead2;
        while(p1) {
            p1 = p1->next;
            ++x;
        }
        while(p2) {
            p2 = p2->next;
            ++y;
        }
        p1 = pHead1;
        while(x > y) {
            p1 = p1->next;
            --x;
        }
        p2 = pHead2;
        while(y > x) {
            p2 = p2->next;
            --y;
        }
        while(p1 && p2) {
            if(p1 == p2) {
                return p1;
            }
            p1 = p1->next;
            p2 = p2->next;
        }
        return nullptr;
    }
};
```

+ 更简洁的做法，交替走

```c++
class Solution {
public:
    ListNode *findFirstCommonNode(ListNode *headA, ListNode *headB) {
        auto p = headA, q = headB;
        while(p != q) {
            if(p) p = p->next;
            else p = headB;
            if (q) q = q->next;
            else q = headA;
        }
        return p;
    }
};
```

### 数字在排序数组中出现的次数

> 统计一个数字在排序数组中出现的次数。

```c++
class Solution {
public:
    int GetNumberOfK(vector<int> data ,int k) {
        if(data.empty()) {
            return 0;
        }
        // lower_bound
        int left = 0, right = data.size();
        while(left < right) {
            int mid = (left + right)/2;
            if(data[mid] < k) {
                left = mid+1;
            }
            else {
                right = mid;
            }
        }
        if(left >= data.size() || data[left] != k) {
            return 0;
        }
        
        int last = left;
        
        // upper_bound
        left = 0, right = data.size();
        while(left < right) {
            int mid = (left + right)/2;
            if(data[mid] <= k) {
                left = mid+1;
            }
            else {
                right = mid;
            }
        }
        
        return right-last;
    }
};
```

### 二叉树的深度

> 输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。

```c++
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    int TreeDepth(TreeNode* pRoot)
    {
        if(!pRoot)
            return 0;
        return 1 + std::max(TreeDepth(pRoot->left), TreeDepth(pRoot->right));
    }
};
```

### 平衡二叉树

> 输入一棵二叉树，判断该二叉树是否是平衡二叉树。

```c++
class Solution {
public:
    int TreeDeepth(TreeNode* pRoot) {
        if(!pRoot)
            return 0;
        int left = TreeDeepth(pRoot->left);
        int right = TreeDeepth(pRoot->right);
        if(left < 0 || right < 0 || abs(left-right) > 1) 
            return -1;
        return std::max(left, right) + 1;
    }
    bool IsBalanced_Solution(TreeNode* pRoot) {
        if(!pRoot) 
            return true;
        return TreeDeepth(pRoot) >= 0;
    }
};
```

### 数组中只出现一次的数字

> 一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

```c++
class Solution {
public:
    void FindNumsAppearOnce(vector<int> data,int* num1,int *num2) {
        int x = 0;
        for(auto& i : data) {
            x ^= i;
        }
        int px = 0;
        while(!(x>>px & 1)) {
            ++px;
        }
        int y = 0;
        x = 0;
        for(auto& i : data) {
            if(i>>px & 1) 
                x ^= i;
            else
                y ^= i;
        }
        *num1 = x;
        *num2 = y;
    }
};
```

### 数组中唯一只出现一次的数字

> 在一个数组中除了一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。你可以假设满足条件的数字一定存在。

```c++
class Solution {
public:
    int findNumberAppearingOnce(vector<int>& nums) {
        int length = nums.size();
        if (length == 0) return -1;
        int bitsum[32] = {0};
        for (int i = 0; i < length; i++){
            int bitmask = 1;
            for (int j = 31; j >= 0; j-- ){
                bitsum[j] += (nums[i] & bitmask)?1 : 0;
                bitmask = bitmask << 1;
            }
        }
        int result = 0;
        for (int i = 0; i < 32; i++) {
            result = result << 1;
            result += bitsum[i]%3;
        }   
        return result;
    }
};
```

### 和为S的两个数字

> 输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。

```c++
class Solution {
public:
    vector<int> FindNumbersWithSum(vector<int> array,int sum) {
        vector<int> ans;
        if(array.size() < 2) {
            return ans;
        }
        int left = 0, right = array.size()-1;
        long long x = sum, y = sum;
        while(left < right) {
            if(array[left] + array[right] == sum) {
                if(array[left]*array[right] < x*y) {
                    x = array[left];
                    y = array[right];
                }
                --right;
            }
            else if(array[left] + array[right] < sum) {
                ++left;
            }
            else {
                --right;
            }
        }
        
        if(x + y == sum) {
            ans.push_back(x);
            ans.push_back(y);
        }
        return ans;
    }
};
```

### 和为S的连续正数序列

> 小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck!

```c++
class Solution {
public: 
    void AddAnswer(vector<vector<int> >& ans, int left, int right) {
        vector<int> temp;
        for(int i = left; i <= right; ++i) {
            temp.push_back(i);
        }
        ans.push_back(temp);
    }
    
    vector<vector<int> > FindContinuousSequence(int sum) {
        vector<vector<int> > ans;
        if(sum <= 0) {
            return ans;
        }
        int left = 1, right = 2, temp_sum = 1 + 2;
        while(left < right) {
            if(temp_sum == sum) {
                AddAnswer(ans, left, right);
                temp_sum -= left;
                ++left;
            }
            else if(temp_sum > sum) {
                temp_sum -= left;
                ++left;
            }
            else {                
                ++right;
                temp_sum += right;
            }
        }
        return ans;
    }
};
```

### 左旋转字符串

> 汇编语言中有一种移位指令叫做循环左移（ROL），现在有个简单的任务，就是用字符串模拟这个指令的运算结果。对于一个给定的字符序列S，请你把其循环左移K位后的序列输出。例如，字符序列S=”abcXYZdef”,要求输出循环左移3位后的结果，即“XYZdefabc”。是不是很简单？OK，搞定它！

```c++
class Solution {
public:
    void reverse(string& str, int left, int right) {
        while(left < right) {
            char temp_c = str[left];
            str[left] = str[right];
            str[right] = temp_c;
            ++left, --right;
        }
    } 
    
    string LeftRotateString(string str, int n) {
        string ans;
        if(str.empty()) {
            return ans;
        }
        n %= str.size();
        reverse(str, 0, str.size()-1);
        reverse(str, str.size()-n, str.size()-1);
        reverse(str, 0, str.size()-n-1);
        return str;
    }
};
```

### 反转单词序列

> 牛客最近来了一个新员工Fish，每天早晨总是会拿着一本英文杂志，写些句子在本子上。同事Cat对Fish写的内容颇感兴趣，有一天他向Fish借来翻看，但却读不懂它的意思。例如，“student. a am I”。后来才意识到，这家伙原来把句子单词的顺序翻转了，正确的句子应该是“I am a student.”。Cat对一一的翻转这些单词顺序可不在行，你能帮助他么？

```c++
class Solution {
public:
    void reverse(string& str, int left, int right) {
        while(left < right) {
            char temp_c = str[left];
            str[left] = str[right];
            str[right] = temp_c;
            ++left, --right;
        }
    } 
    
    string ReverseSentence(string str) {
        if(str.empty()) {
            return str;
        }
        reverse(str, 0, str.size()-1);
        int left = 0, right = 0;
        while(left < str.size()) {
            if(str[left] == ' ') {
                ++left;
                right = left;
            }
            else if(right == str.size() || str[right] == ' ') {
                reverse(str, left, right-1);
                left = ++right;
            }
            else {
                ++right;
            }
        }
        return str;
    }
};
```

### 扑克牌顺子

> LL今天心情特别好,因为他去买了一副扑克牌,发现里面居然有2个大王,2个小王(一副牌原本是54张^_^)...他随机从中抽出了5张牌,想测测自己的手气,看看能不能抽到顺子,如果抽到的话,他决定去买体育彩票,嘿嘿！！“红心A,黑桃3,小王,大王,方片5”,“Oh My God!”不是顺子.....LL不高兴了,他想了想,决定大\小 王可以看成任何数字,并且A看作1,J为11,Q为12,K为13。上面的5张牌就可以变成“1,2,3,4,5”(大小王分别看作2和4),“So Lucky!”。LL决定去买体育彩票啦。 现在,要求你使用这幅牌模拟上面的过程,然后告诉我们LL的运气如何， 如果牌能组成顺子就输出true，否则就输出false。为了方便起见,你可以认为大小王是0。

```c++
class Solution {
public:
    bool IsContinuous( vector<int> numbers ) {
        if(numbers.size() < 2) {
            return !numbers.empty();
        }
        sort(numbers.begin(), numbers.end());
        int x = 0, y = 0;
        for(int i = 0; i < numbers.size(); ++i) {
            if(!numbers[i]) {
                ++x;
            }
            else if(i > 0 && numbers[i-1]) {
                if(numbers[i] == numbers[i-1]) {
                    return false;
                }
                y += numbers[i] - numbers[i-1] - 1;
            }
        }
        return x >= y;
    }
};
```

### 圆圈中最后剩下的数

> 每年六一儿童节,牛客都会准备一些小礼物去看望孤儿院的小朋友,今年亦是如此。HF作为牛客的资深元老,自然也准备了一些小游戏。其中,有个游戏是这样的:首先,让小朋友们围成一个大圈。然后,他随机指定一个数m,让编号为0的小朋友开始报数。每次喊到m-1的那个小朋友要出列唱首歌,然后可以在礼品箱中任意的挑选礼物,并且不再回到圈中,从他的下一个小朋友开始,继续0...m-1报数....这样下去....直到剩下最后一个小朋友,可以不用表演,并且拿到牛客名贵的“名侦探柯南”典藏版(名额有限哦!!^_^)。请你试着想下,哪个小朋友会得到这份礼品呢？(注：小朋友的编号是从0到n-1)

```c++
class Solution {
public:
    int LastRemaining_Solution(int n, int m)
    {
        if(n < 1 || m < 1) {
            return -1;
        }
        int last = 0;
        for(int i = 2; i <= n; ++i) {
            last = (last+m)%i;
        }
        return last;
    }
};
```

### 求1+2+3+...+n

> 求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

```c++
class Solution {
public:  
    typedef int (*func)(int);
    
    static int Sum_Solution_Zero(int n) {
        return 0;
    }
    static int Sum_Solution(int n) {
        static func f[2] = {Sum_Solution_Zero, Sum_Solution};
        return f[!!n](n-1) + n;
    }
};
```

### 不用加减乘除做加法

> 写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。

```c++
class Solution {
public:
    int Add(int num1, int num2)
    {
        while(num2) {
            int sum = num1 ^ num2;
            int carry = (num1 & num2) << 1;
            num1 = sum;
            num2 = carry;
        }
        return num1;
    }
};
```

### 把字符串转换为整数

> 将一个字符串转换成一个整数(实现Integer.valueOf(string)的功能，但是string不符合数字要求时返回0)，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0。

```c++
class Solution {
public:
    int StrToInt(string str) {
        if(str.empty()) {
            return 0;
        }
        int ans = 0, pos = 0, neg = 1;
        while(pos < str.size() && str[pos] == ' ') ++pos;
        if(pos < str.size()) {
            if(str[pos] == '-') {
                neg = -1;
                ++pos;
            }
            else if(str[pos] == '+') {
                ++pos;
            }
        }  
        while(pos < str.size() && str[pos] >= '0' && str[pos] <= '9') {
            ans = ans*10 + str[pos] - '0';
            ++pos;
        }
        return pos == str.size() ? neg*ans : 0;
    }
};
```

### 数组中重复的数字

> 在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。

```c++
class Solution {
public:
    // Parameters:
    //        numbers:     an array of integers
    //        length:      the length of array numbers
    //        duplication: (Output) the duplicated number in the array number
    //        Return value:  true if the input is valid, and there are some duplications 
    //                       in the array number otherwise false
    bool duplicate(int numbers[], int length, int* duplication) {
        int ans = -1, start = 0;
        while(start < length) {
            while(numbers[start] < length && numbers[start] != start) {
                int temp = numbers[start];
                if(numbers[start] == numbers[temp]) {
                    *duplication = numbers[start];
                    return true;
                }
                numbers[start] = numbers[temp];
                numbers[temp] = temp;
            }
            if(numbers[start] >= length) {
                return false;
            }
            ++start;
        }
        return false;
    }
    // 如果是总共 n 个数，所有的数都是在 1~n-1 的范围内才可以用二分
    // 例如 2 1 3 1 4， 假定数字范围为 0 ~ 4, 则第一次二分出 0~2 有3个，那么下一步搜索 3~4，不会找到1
};
```

### 构建乘积数组

> 给定一个数组`A[0,1,...,n-1]`,请构建一个数组`B[0,1,...,n-1]`,其中B中的元素`B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]`。不能使用除法。

```c++
class Solution {
public:
    vector<int> multiply(const vector<int>& A) {
        vector<int> B;
        if(A.size() < 2) {
            return A;
        }
        B.push_back(1);
        for(int i = 1; i < A.size(); ++i) {
            B.push_back(B.back()*A[i-1]);
        }
        int last = A.back();
        for(int i = A.size()-2; i >= 0; --i) {
            B[i] *= last;
            last *= A[i];
        }
        return B;
    }
};
```

### 正则表达式匹配

> 请实现一个函数用来匹配包括`'.'`和`'*'`的正则表达式。模式中的字符'.'表示任意一个字符，而`'*'`表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串`"aaa"`与模式`"a.a"`和`"ab*ac*a"`匹配，但是与`"aa.a"`和`"ab*a"`均不匹配

```c++
class Solution {
public:
    bool match(char* str, char* pattern)
    {
        int n = strlen(str), m = strlen(pattern);
        bool dp[2][m+1];
        dp[n%2][m] = true;
        for(int i = n; i >= 0; --i) {
            for(int j = m; j >= 0; --j) {
                if(i == n && j == m) continue;
                bool is_match = i < n && j < m && (str[i] == pattern[j] || pattern[j] == '.');
                if(j < m-1 && pattern[j+1] == '*') {
                    dp[i%2][j] = dp[i%2][j+2] || (is_match && dp[(i+1)%2][j]);
                }
                else {
                    dp[i%2][j] = is_match && dp[(i+1)%2][j+1];
                }
            }
        }
        return dp[0][0];
    }
};
```

### 表示数值的字符串

> 请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串`"+100","5e2","-123","3.1416"`和`"-1E-16"`都表示数值。 但是`"12e","1a3.14","1.2.3","+-5"`和`"12e+4.3"`都不是。

```c++
class Solution {
public:
    bool isNumeric(char* string)
    {
        int n = strlen(string);
        bool has_e = false, has_dot = false;
        for(int i = 0; i < n; ++i) {
            if(string[i] == 'e' || string[i] == 'E') {
                if(has_e || i == n-1) 
                    return false;
                has_e = true;
            }
            else if(string[i] == '.') {
                if(has_dot || has_e) return false;
                has_dot = true;
            }
            else if(string[i] == '+' || string[i] == '-') {
                if(i > 0 && !has_e) return false;
                if(has_e && string[i-1] != 'e' && string[i-1] != 'E') return false;
            }
            else if(string[i] > '9' || string[i] < '0') {
                return false;
            }
        }
        return true;
    }
};
```

### 字符流中第一个不重复的字符

> 请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符`"go"`时，第一个只出现一次的字符是`"g"`。当从该字符流中读出前六个字符`“google"`时，第一个只出现一次的字符是`"l"`。

```c++
class Solution
{
public:
    Solution() {
        memset(char_pos, 0, sizeof(int)*256);
        cnt = 1;
    }
  //Insert one char from stringstream
    void Insert(char ch)
    {
        if(char_pos[ch] > 0) 
            char_pos[ch] = -1;
        else if(char_pos[ch] == 0) {
            char_pos[ch] = cnt++;
        }
    }
  //return the first appearence once char in current stringstream
    char FirstAppearingOnce()
    {
        int min_pos = INT_MAX;
        char ans;
        for(int i = 0; i < 256; ++i) {
            if(char_pos[i] > 0 && min_pos > char_pos[i]) {
                min_pos = char_pos[i];
                ans = (i+'\0'); // or ` ans = i ` is ok
            } 
        }
        if(min_pos == INT_MAX) 
            return '#';
        return ans;
    }
private:
    int char_pos[256];
    int cnt;
};
```

### 链表环中的入口节点

> 给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

```c++
/*
struct ListNode {
    int val;
    struct ListNode *next;
    ListNode(int x) :
        val(x), next(NULL) {
    }
};
*/
class Solution {
public:
    ListNode* FindMeetingNode(ListNode* pHead)
    {
        ListNode* pslow = pHead;
        ListNode* pfast = pHead->next;
        while(pslow && pfast && pslow != pfast) {
            pslow = pslow->next;
            pfast = pfast->next;
            if(pfast) {
                pfast = pfast->next;
            }
        }
        if(pslow && pfast && pslow == pfast) {
            return pslow;
        }
        return nullptr;
    }
    
    ListNode* EntryNodeOfLoop(ListNode* pHead)
    {
        if(!pHead || pHead->next == nullptr) {
            return nullptr;
        }
        ListNode* meeting_node = FindMeetingNode(pHead);
        if(!meeting_node) {
            return nullptr;
        }
        int num_in_loop = 1;
        ListNode* temp_node = meeting_node->next;
        while(temp_node != meeting_node) {
            temp_node = temp_node->next;
            ++num_in_loop;
        }
        temp_node = pHead;
        while(num_in_loop--) {
            temp_node = temp_node->next;
        }
        ListNode* temp_node2 = pHead;
        while(temp_node != temp_node2) {
            temp_node = temp_node->next;
            temp_node2 = temp_node2->next;;
        }
        return temp_node;
    }
};
```

### 删除链表中重复的节点

> 在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

```c++
/*
struct ListNode {
    int val;
    struct ListNode *next;
    ListNode(int x) :
        val(x), next(NULL) {
    }
};
*/
class Solution {
public:
    ListNode* deleteDuplication(ListNode* pHead)
    {
        if(!pHead || pHead->next == nullptr) {
            return pHead;
        }
        ListNode* temp_head = new ListNode(-1);
        temp_head->next = pHead;
        ListNode* first = temp_head;
        ListNode* second = temp_head->next; 
        while(second) {
            if(second->next && second->val == second->next->val) {
                while(second->next && second->val == second->next->val) {
                    second = second->next;
                }
                first->next = second->next;
            } 
            else {
                first = first->next;
            }
            second = second->next;
        }
        return temp_head->next;
    }
};
```

### 二叉树的下一个节点

> 给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

```c++
/*
struct TreeLinkNode {
    int val;
    struct TreeLinkNode *left;
    struct TreeLinkNode *right;
    struct TreeLinkNode *next;
    TreeLinkNode(int x) :val(x), left(NULL), right(NULL), next(NULL) {
        
    }
};
*/
class Solution {
public:
    TreeLinkNode* GetNext(TreeLinkNode* pNode)
    {
        if(!pNode) {
            return nullptr;
        }
        if(pNode->right) {
            pNode = pNode->right;
            while(pNode && pNode->left) {
                pNode = pNode->left;
            }
            return pNode;
        }
        if(pNode->next) {
            if(pNode->next->left == pNode) {
                return pNode->next;
            }
            TreeLinkNode* fa = pNode->next;
            while(pNode && fa && fa->left != pNode) {
                pNode = fa;
                fa = fa->next;
            }
            if(fa && fa->left == pNode) {
                return fa;
            }
            return nullptr;
        }
        return nullptr;
    }
};
```

### 对称的二叉树

> 请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    bool isSymmetricalCore(TreeNode* pleft, TreeNode* pright) {
        if(!pleft || !pright) {
            return !pleft && !pright;
        }
        if(pleft->val == pright->val) {
            return isSymmetricalCore(pleft->left, pright->right)
                   && isSymmetricalCore(pleft->right, pright->left);
        }
        return false;
    }
    
    bool isSymmetrical(TreeNode* pRoot)
    {
        if(!pRoot) {
            return true;
        }
        return isSymmetricalCore(pRoot->left, pRoot->right);
    }

};
```

### 按之字形顺序打印二叉树

> 请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。

```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    vector<vector<int> > Print(TreeNode* pRoot) {
        vector<vector<int> > ans;
        if(!pRoot) {
            return ans;
        }
        deque<TreeNode*> que[2];
        int index = 0;
        que[index].push_back(pRoot);
        vector<int> line;
        while(!que[index].empty()) {
            TreeNode* head = nullptr;
            if(index) {
                head = que[index].back();
                line.push_back(head->val);    
                que[index].pop_back();
                if(head->right) {
                    que[1-index].push_back(head->right);
                }
                if(head->left) {
                    que[1-index].push_back(head->left);
                }
            }
            else {
                head = que[index].back();
                line.push_back(head->val);    
                que[index].pop_back();
                if(head->left) {
                    que[1-index].push_back(head->left);
                }
                if(head->right) {
                    que[1-index].push_back(head->right);
                }
            }

            if(que[index].empty()) {
                index = 1 - index;
                ans.push_back(line);
                line.clear();
            }
        }
        return ans;
    }
    
};
```

### 把二叉树打印成多行

> 从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。

```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
        vector<vector<int> > Print(TreeNode* pRoot) {
            vector<vector<int> > ans;
            if(!pRoot) {
                return ans;
            }
            queue<TreeNode*> que;
            que.push(pRoot);
            int cnt = 1;
            vector<int> temp_ans;
            while(!que.empty()) {   
                TreeNode* temp_node = que.front();
                temp_ans.push_back(temp_node->val);
                que.pop();
                if(temp_node->left) {
                    que.push(temp_node->left);
                }
                if(temp_node->right) {
                    que.push(temp_node->right);
                }
                --cnt;
                if(cnt == 0) {
                    ans.push_back(temp_ans);
                    temp_ans.clear();
                    cnt = que.size();
                }
            }
            return ans;
        }
    
};
```

### 序列化二叉树

> 请实现两个函数，分别用来序列化和反序列化二叉树

```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    void SerializeCore(TreeNode *root, string& ans) {
        if(!root) {
            ans += "$,";
            return;
        }
        ans += to_string(root->val) + ",";
        SerializeCore(root->left, ans);
        SerializeCore(root->right, ans);
    }
    
    char* Serialize(TreeNode *root) {    
        string ans;
        SerializeCore(root, ans);
        char *ansp = new char[ans.length()];
        memcpy(ansp, ans.c_str(), ans.length());
        return ansp;
    }
    
    bool ReadNumber(char *str, int& pos, int& number) {
        number = 0;
        bool has_num = false;
        while(str[pos]) {
            if(str[pos] >= '0' && str[pos] <= '9') {
                number = number*10 + (str[pos] - '0');
                has_num = true;
                ++pos;
            }
            else if(str[pos] == ',') {
                ++pos;
                break;
            }
            else if(str[pos] == '$') {
                ++pos;
            }
        }
        return has_num;
    }
    
    void DeserializeCore(TreeNode** root, char *str, int& pos) {
        int number;
        if(ReadNumber(str, pos, number)) {
            (*root) = new TreeNode(number);
            DeserializeCore(&((*root)->left), str, pos);
            DeserializeCore(&((*root)->right), str, pos);
        }
    }
    
    TreeNode* Deserialize(char *str) {
        TreeNode* ans = nullptr;
        if(!str || str[0] == '$') {
            return ans;
        }
        int pos = 0;
        DeserializeCore(&ans, str, pos);
        return ans;
    }

};
```

### 二叉搜索树的第k个节点

> 给定一棵二叉搜索树，请找出其中的第k小的结点。例如，（5，3，7，2，4，6，8）中，按结点数值大小顺序第三小结点的值为4。

```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    TreeNode* KthNodeCore(TreeNode* pRoot, int& k) {
        TreeNode* target = nullptr;
        if(pRoot->left) {
            target = KthNodeCore(pRoot->left, k);
        }
        if(target == nullptr) {
            if(k == 1) {
                target = pRoot;
            }
            --k;
        }
        if(target == nullptr && pRoot->right) {
            target = KthNodeCore(pRoot->right, k);
        }
        return target;
    }
    
    TreeNode* KthNode(TreeNode* pRoot, int k)
    {
        if(!pRoot || !k) {
            return nullptr;
        }
        return KthNodeCore(pRoot, k);
    }
};
```

### 数据流中的中位数

> 如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。

```c++
class Solution {
public:
    void Insert(int num)
    {
        if(min_half_.size() <= max_half_.size()) {
            if(max_half_.size() > 0 && num < max_half_.top()) {
                max_half_.push(num);
                num = max_half_.top();
                max_half_.pop();
            }
            min_half_.push(num);
        }
        else {
            if(min_half_.size() > 0 && num > min_half_.top()) {
                min_half_.push(num);
                num = min_half_.top();
                min_half_.pop();
            }
            max_half_.push(num);
        }
    }

    double GetMedian()
    { 
        if((min_half_.size() + max_half_.size())%2) {
            return min_half_.top();
        }
        else if(!min_half_.empty()) {
            return (min_half_.top() + max_half_.top())/2.0;
        }
        return 0.0;
    }
    
private:
    priority_queue<int, std::vector<int> ,std::greater<int> > min_half_;
    priority_queue<int> max_half_;

};
```

### 滑动窗口的最大值

> 给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。

```c++
class Solution {
public:
    vector<int> maxInWindows(const vector<int>& num, unsigned int size)
    {
        vector<int> ans;
        if(size == 0 || num.empty()) {
            return ans;
        }
        deque<int> que; // 表示下标，如果直接用数字 7 2 5 6 1, size = 3 时会有问题。
        for(int i = 0; i < num.size(); ++i) {
            while(!que.empty() && num[que.back()] <= num[i]) {
                que.pop_back();
            }
            while(!que.empty() && i-que.front() > size-1) {
                que.pop_front();
            }
            que.push_back(i);
            if(i >= size-1) {
                ans.push_back(num[que.front()]);
            }
        }
        return ans;
    }
};
```

### 矩阵中的路径

> 请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 a b c e s f c s a d e e 这样的3 X 4 矩阵中包含一条字符串"bcced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。

```c++
class Solution {
public:
    bool hasPathCore(char* matrix, int rows, int cols, char* str, 
                     bool* visited, int x, int y, int pos) {
        if(str[pos] == '\0') {
            return true;
        }

        bool hasPath = false;
        if(x >= 0 && y >= 0 && x < rows && y < cols 
           && matrix[x*cols+y] == str[pos] && !visited[x*cols+y]) {
            ++pos;
            visited[x*cols+y] = true;
            hasPath = hasPathCore(matrix, rows, cols, str, visited, x+1, y, pos)
                     || hasPathCore(matrix, rows, cols, str, visited, x, y+1, pos)
                     || hasPathCore(matrix, rows, cols, str, visited, x, y-1, pos)
                     || hasPathCore(matrix, rows, cols, str, visited, x-1, y, pos);
            if(!hasPath) {
                --pos;
                visited[x*cols+y] = false;
            }
            
        }
        return hasPath;
    }
    
    bool hasPath(char* matrix, int rows, int cols, char* str)
    {
        if(!matrix || !str) {
            return false;
        }
        bool* visited = new bool[rows*cols];
        memset(visited, 0, rows*cols);
        for(int i = 0; i < rows; ++i) {
            for(int j = 0; j < cols; ++j) {
                if(hasPathCore(matrix, rows, cols, str, visited, i, j, 0)) {
                    return true;
                }
            }
        }
        delete[] visited;
        return false;
    }
};
```

### 机器人的运动范围

> 地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？

```c++
class Solution {
public:
    int digitSum(int x, int y) {
        int ans = 0;
        while(x) {
            ans += (x%10);
            x /= 10;
        }
        while(y) {
            ans += (y%10);
            y /= 10;
        }
        return ans;
    }
    
    int movingCountCore(int threshold, int rows, int cols, int x, int y, bool* visited) 
    {
        int count = 0;
        if(x >= 0 && y >= 0 && x < rows && y < cols 
           && digitSum(x, y) <= threshold && !visited[x*cols+y]) {
            visited[x*cols+y] = true;
            count = 1 + movingCountCore(threshold, rows, cols, x-1, y, visited)
                     + movingCountCore(threshold, rows, cols, x, y-1, visited)
                     + movingCountCore(threshold, rows, cols, x+1, y, visited)
                     + movingCountCore(threshold, rows, cols, x, y+1, visited);
        }
        return count;
    }
    
    int movingCount(int threshold, int rows, int cols)
    {
        if(threshold < 0 || rows <= 0 || cols <= 0) {
            return 0;
        }
        bool* visited = new bool[rows*cols];
        memset(visited, 0, rows*cols);
        int count = movingCountCore(threshold, rows, cols, 0, 0, visited);
        delete[] visited;
        return count;
    }
};
```

