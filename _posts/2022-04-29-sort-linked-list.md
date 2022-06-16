---
layout: post
title: 链表排序
categories: [leetcode, 链表]
description: 链表排序
keywords: leetcode, 链表，排序
---

链表排序类似数组排序，可以采用归并排序、快速排序等常见排序算法，归并排序需要遍历链表找到中点，快速排序可选取第一个节点作为比较基准，节点交换时，一种方法交换节点的值，另一种方法是将左边和右边分别形成一个链表，这种方法需要处理链表两端的拼接，比值交换复杂。

下边直接上代码。

## 归并排序

```c++
#include <iostream>
#include <memory>
#include <vector>

using namespace std;

struct Node {
 int data;
 Node* next;
 Node(int data_=0, Node* next_=0): data(data_), next(next_) {}
 ~Node() {}
};

void listDelete(Node* head) {
 while (head) {
  Node* p = head;
  head = head->next;
  delete p;
 }
 head = 0;
}

void listPrint(Node* head) {
 while (head) {
  cout << head->data;
  head = head->next;
  if (head) cout << "->";
 }
 cout << endl;
}

Node* listMerge(Node* head1, Node* head2) {
 Node* dummy = new Node();
 Node* cur = dummy;
 while (head1 && head2) {
  if (head1->data < head2->data) {
   cur->next = head1;
   head1 = head1->next;
  } else {
   cur->next = head2;
   head2 = head2->next;
  }
  cur = cur->next;
 }
 if (head1) cur->next = head1;
 if (head2) cur->next = head2;
 cur = dummy->next;
 delete dummy;
 return cur;
}

Node* listSort(Node* head) {
 // 递归终止条件
 if (!head || !head->next) return head;
 // 快慢指针找中点
 Node* dummy = new Node(0, head);
 Node *slow = dummy, *fast = head;
 while (fast && fast->next) {
  slow = slow->next;
  fast = fast->next->next;
 }
 // slow->next 即是分割后后一个链表的头
 Node* head2 = slow->next;
 slow->next = 0;
 delete dummy;
 head = listSort(head);
 head2 = listSort(head2);
 return listMerge(head, head2);
}

int main(int argc, char** argv) {
 Node* dummy = new Node();
 Node* cur = dummy;
 int data;
 while (cin >> data) {
  cur->next = new Node(data);
  cur = cur->next;
 }
 Node* head = dummy->next;
 delete dummy;
 cout << "list: ";
 listPrint(head);
 head = listSort(head);
 cout << "sorted list: ";
 listPrint(head);
 listDelete(head);
 return 0;
}
```

## 快速排序

单链表：

```c++
#include <iostream>
#include <memory>
#include <vector>

using namespace std;

struct Node
{
    int data;
    Node *next;
    Node(int data_ = 0, Node *next_ = 0) : data(data_), next(next_) {}
    ~Node() {}
};

class MyList
{
private:
    Node *head_;

public:
    MyList() : head_(nullptr) {}
    ~MyList()
    {
        while (head_)
        {
            Node *p = head_;
            head_ = head_->next;
            delete p;
        }
    }
    void addNode(int data)
    {
        head_ = new Node(data, head_);
    }
    void quickSort1()
    {
        quickSort1_(head_, nullptr);
    }
    void quickSort2()
    {
        Node *dummy = new Node(0, head_);
        quickSort2_(dummy, head_, nullptr);
        head_ = dummy->next;
        // cout << "quickSort2 head: " << head_->data << endl;
        delete dummy;
    }
    void print() const
    {
        print_(head_);
    }

private:
    void print_(Node *head) const
    {
        cout << "MyList: ";
        Node *p = head;
        while (p)
        {
            cout << p->data;
            p = p->next;
            if (p)
                cout << "->";
        }
        cout << endl;
    }
    // 快排1：交换链表节点的值
    // 左闭右开区间 [left, right)
    void quickSort1_(Node *left, Node *right)
    {
        // 递归终止条件
        if (left == right || left->next == right)
            return;
        Node *pre = left;
        Node *x = left;
        Node *i = left->next;
        Node *j = left->next;
        while (j != right)
        { // right 开区间
            if (j->data < x->data)
            {
                swap(i->data, j->data);
                pre = i;
                i = i->next;
            }
            j = j->next;
        }
        // x 在最左边，i是第一个大于x的，pre是i前一个元素，小于x
        swap(pre->data, x->data);
        quickSort1_(left, pre);
        quickSort1_(pre->next, right);
    }
    // 快排2：比基数节点x小的和大的节点分别组成一个链表
    // 左闭右开区间 [left, right)
    void quickSort2_(Node *preLeft, Node *left, Node *right)
    {
        // 递归终止条件
        if (left == right || left->next == right)
            return;
        Node *low = left;   // low 链表末尾接基数节点left
        Node *high = right; // high 链表末尾接right
        Node *cur = left->next;
        while (cur != right)
        {
            Node *p = cur->next;
            ;
            if (cur->data < left->data)
            {
                cur->next = low;
                low = cur;
            }
            else
            {
                cur->next = high;
                high = cur;
            }
            cur = p;
        }
        preLeft->next = low;
        left->next = high; // 基数节点left后接high链表
        quickSort2_(preLeft, low, left);
        quickSort2_(left, high, right);
    }
};

int main(int argc, char **argv)
{
    vector<int> nums = {5, 4, 3, 2, 1};
    auto myList = make_unique<MyList>();
    // int data;
    // while (cin >> data) {
    //   myList->addNode(data);
    // }
    for (auto num : nums)
    {
        myList->addNode(num);
    }
    myList->print();
    myList->quickSort2();
    myList->print();
    return 0;
}
```

双向链表：

```c++
#include <iostream>
#include <vector>
#include <memory>

using namespace std;

struct Node
{
    int data;
    Node *next;
    Node *prev;

    Node(int data_ = 0, Node *next_ = nullptr, Node *prev_ = nullptr)
        : data(data_), next(next_), prev(prev_) {}

    ~Node() {}
};

class DList
{
private:
    Node *head_;
    Node *tail_;

public:
    DList() : head_(new Node()), tail_(new Node())
    {
        head_->next = tail_;
        tail_->prev = head_;
    }
    ~DList()
    {
        while (head_)
        {
            Node *p = head_;
            head_ = head_->next;
            delete p;
        }
    }
    void addToTail(int data)
    {
        Node *p = new Node(data);
        p->next = tail_;
        p->prev = tail_->prev;
        tail_->prev->next = p;
        tail_->prev = p;
    }
    void addToHead(int data)
    {
        Node *p = new Node(data);
        p->next = head_->next;
        p->prev = head_;
        head_->next->prev = p;
        head_->next = p;
    }
    void sort()
    {
        if (head_->next == tail_)
            return;
        quickSort_(head_->next, tail_->prev);
    }
    void print() const
    {
        cout << "DList: ";
        Node *cur = head_->next;
        while (cur != tail_)
        {
            cout << cur->data;
            cur = cur->next;
            if (cur != tail_)
                cout << "->";
        }
        cout << endl;
    }
    void rprint() const
    {
        cout << "reversed DList: ";
        Node *cur = tail_->prev;
        while (cur != head_)
        {
            cout << cur->data;
            cur = cur->prev;
            if (cur != head_)
                cout << "->";
        }
        cout << endl;
    }

private:
    // 交换节点的值
    // [low, high] 闭区间
    void quickSort_(Node *low, Node *high)
    {
        // cout << "qs: " << low->data << " " << high->data << endl;
        // 递归终止条件
        if (low == high)
            return;
        Node *x = high;
        Node *i = low;
        Node *j = low;
        while (j != high)
        {
            if (j->data < x->data)
            {
                swap(i->data, j->data);
                i = i->next;
            }
            j = j->next;
        }
        swap(i->data, x->data);
        // 注意这里需要排除 i 为 low 的或 high 情况
        if (i != low)
            quickSort_(low, i->prev);
        if (i != high)
            quickSort_(i->next, high);
    }
};

int main(int argc, char **argv)
{
    vector<int> nums = {5, 4, 3, 2, 1};
    auto myList = make_unique<DList>();
    // int data;
    // while (cin >> data) {
    //	myList->addNode(data);
    // }
    for (auto num : nums)
    {
        myList->addToTail(num);
    }
    myList->print();
    myList->sort();
    myList->print();
    myList->rprint();
    cout << "end" << endl;
    return 0;
}
```
