---
marp: false
theme: gaia
footer: '2023/7/14'
paginate: true
size: 16:9
math: katex
---
# 静态内存
1. 局部`static`对象
2. 类`static`数据成员
3. 定义在任何函数体之外的变量

`static`对象在**第一次使用**前分配，在**程序结束**时销毁
全局对象在**程序启动**时分配，在**程序结束**时销毁

---

# 栈内存
函数内的非`static`对象。

**定义时**分配，**离开其定义的程序块**时销毁

---
# 堆
动态分配的对象

**代码显式销毁**

---

# 动态内存管理
`new`：在动态内存中为对象分配空间并返回一个指向该对象的指针，可对对象进行初始化。
`delete`：接受一个动态对象的指针，销毁该对象，并释放与之关联的内存。

释放一块并非`new`分配的内存，或将相同的指针值**释放多次**，其行为是**未定义**的。

## 内存泄露
忘记释放内存

## 使用空悬指针
使用已经释放的内存。

`delete`之后要将指针**置空**。但对其他还指向该内存的指针无用。

---

# 智能指针
负责**自动释放**所指向的对象。定义在头文件`memory`中。
1. `shared_ptr`:允许多个指针指向同一个对象
2. `unique_ptr`:“独占”所指向的对象
3. `weak_ptr`:伴随类，它是一种弱引用，指向`shared_ptr`所管理的对象。

---

## `shared_ptr`类
必须提供指针可以指向的类型
```
shared_ptr<T> p;//模板，可以指向类型T的指针
```
### `make_shared`函数
**最安全的分配和使用动态内存的方法**
```
shared_ptr<T> p = make_shared<T>(/*用该参数构造给定类型的对象*/);

//利用auto简化
auto p = make_shared<T>(/*用该参数构造给定类型的对象*/);
```

---

### 引用计数
计数器递增：
1. 用该`shared_ptr`初始化另一个`shared_ptr`
2. 将该`shared_ptr`作为参数传递给一个函数
3. 作为函数的返回值

```
shared_ptr(const shared_ptr& _Other) noexcept { // construct shared_ptr object that owns same resource as _Other
    this->_Copy_construct_from(_Other);
}

void _Copy_construct_from(const shared_ptr<_Ty2>& _Other) noexcept {
    // implement shared_ptr's (converting) copy ctor
    _Other._Incref();

    _Ptr = _Other._Ptr; //element_type* _Ptr{nullptr};
    _Rep = _Other._Rep; //_Ref_count_base* _Rep{nullptr};
}

void _Incref() noexcept { // increment use count
    _MT_INCR(_Uses);//实现数的原子性加
}
```

---

计数器递减：
1. 被赋予新值
2. 被销毁

```
shared_ptr& operator=(const shared_ptr& _Right) noexcept {
    shared_ptr(_Right).swap(*this);//用右边的创建了一个临时对象，然后跟左边的交换
    return *this;
}

~shared_ptr() noexcept { // release resource
    this->_Decref();
}

void _Decref() noexcept { // decrement reference count
    if (_Rep) {
        _Rep->_Decref();
    }
}

//当shared_ptr计数器变为0时，就会自动释放自己管理的对象。
//_Ref_count_base的
void _Decref() noexcept { // decrement use count
    if (_MT_DECR(_Uses) == 0) {//实现数的原子性减
        _Destroy();//释放内存
        _Decwref();
    }
}

void _Decwref() noexcept { // decrement weak reference count
    if (_MT_DECR(_Weaks) == 0) {
        _Delete_this();//释放计数器_Ref_count_base
    }
}
```
如果将`shared_ptr`存放于一个容器中，而后不再需要全部元素，而只使用其中一部分，要记得用`erase`删除不再需要的那些元素。

**不要混合使用普通指针和智能指针**：智能指针把内存释放了，普通指针成为空悬指针

**不要使用`get`初始化另一个智能指针或为智能指针赋值**：用`get`返回的指针初始化或赋值另一个智能指针，两个智能指针是**相互独立**的。
```
explicit shared_ptr(_Ux* _Px) { // construct shared_ptr object that owns _Px
    if constexpr (is_array_v<_Ty>) {
        _Setpd(_Px, default_delete<_Ux[]>{});
    } else {
        _Temporary_owner<_Ux> _Owner(_Px);
        _Set_ptr_rep_and_enable_shared(_Owner._Ptr, new _Ref_count<_Ux>(_Owner._Ptr));
        _Owner._Ptr = nullptr;
    }
}
```
---

## `unique_ptr`类
**不支持拷贝和赋值**，可以利用`release`或`reset`将指针的所有权转移
```
unique_ptr<string> p2(p1.release()); //p1转移给p2

p2.reset(p3.release());//p2放弃了原来的，重新指向p3
```
```
_Compressed_pair<_Dx, pointer> _Mypair;//_Dx是删除器

pointer release() noexcept {
    return _STD exchange(_Mypair._Myval2, nullptr);
}

void reset(pointer _Ptr = nullptr) noexcept {
    pointer _Old = _STD exchange(_Mypair._Myval2, _Ptr);
    if (_Old) {
        _Mypair._Get_first()(_Old);//调用删除器
    }
}

```
可以拷贝或赋值一个即将被销毁的`unique_ptr`：函数返回一个`unique_ptr`或局部对象的拷贝。

---

### 向`unique_ptr`传递删除器
**尖括号中：**
```
unique_ptr<objT, delT> p(new objT,fcn); //注意delT是一个函数指针类型
```

---

## `weak_ptr`类
**不控制所指向对象生存期**，指向一个`shared_ptr`管理的对象。

不改变`shared_ptr`的引用计数。由于对象可能不存在，所以不能使用`weak_ptr`直接访问对象，必须调用`lock`(检查所指对象是否存在，存在返回一个指向它的`shared_ptr`，否则返回空`shared_ptr`)

```
weak_ptr(const shared_ptr<_Ty2>& _Other) noexcept {
    this->_Weakly_construct_from(_Other); // shared_ptr keeps resource alive during conversion
}

void _Weakly_construct_from(const _Ptr_base<_Ty2>& _Other) noexcept { // implement weak_ptr's ctors
    if (_Other._Rep) {
        _Ptr = _Other._Ptr;
        _Rep = _Other._Rep;
        _Rep->_Incwref();
    } else {
        _STL_INTERNAL_CHECK(!_Ptr && !_Rep);
    }
}

~weak_ptr() noexcept {
    this->_Decwref();
}


_NODISCARD shared_ptr<_Ty> lock() const noexcept { // convert to shared_ptr
    shared_ptr<_Ty> _Ret;
    (void) _Ret._Construct_from_weak(*this);
    return _Ret;
}

bool _Construct_from_weak(const weak_ptr<_Ty2>& _Other) noexcept {
    // implement shared_ptr's ctor from weak_ptr, and weak_ptr::lock()
    if (_Other._Rep && _Other._Rep->_Incref_nz()) {
        _Ptr = _Other._Ptr;
        _Rep = _Other._Rep;
        return true;
    }

    return false;
}
```

---

# 动态数组
一般使用标准库容器，可以使用默认版本的拷贝、赋值和析构操作。分配动态数组的类**必须定义自己版本的操作**，在拷贝、赋值以及销毁对象时管理所关联的内存。

---

## `new`和数组
```
int *pia = new int[get_size()];

//或数组类型别名
typedef int arrT[42];
int *p = new arrT;
```
方括号中的大小必须是整型，但**不必是常量**

**当用`new`分配一个数组时，我们并未得到一个数组类型的对象，而是得到一个数组元素类型的指针**，不能对其使用`begin`和`end`，也不能使用范围`for`语句。

**动态分配一个空数组是合法的**，返回一个合法的非空指针（类似尾后指针）

---

## 释放动态数组
```
//对使用了类型别名分配的动态数组也必须使用[]
delete [] pa;//忽略方括号其行为是未定义的。
            
```
**按逆序销毁**

---

## 智能指针和动态数组
用`unique_ptr`管理动态数组：
```
unique_ptr<int[]> up(new int[10]);
up.release();//自动用delete[]销毁其指针
```
当用`unique_ptr`指向一个数组时，**不能**使用点和箭头成员运算符。**可以**使用下标运算符来访问数组中的元素。

---

`shared_ptr`不直接支持管理动态数组，必须提供自己定义的删除器。
```
shared_ptr<int> sp(new int[10], [](int *p){ delete [] p; });
```
访问数组中的元素，必须用`get`获取一个内置指针，然后用它来访问数组元素。

---

# `allocator`类
定义在`memory`头文件中，将内存分配和对象构造分离开来。（将内存分配和对象构造组合在一起可能会导致不必要的浪费）

```
//同样是一个模板：
allocator<string> alloc;
auto const p = alloc.allocate(n);//分配n个未初始化的string

//在给定位置构造一个元素：
auto q = p;//q指向最后构造的元素之后的位置
alloc.construct(q++);
alloc.construct(q++, 10, 'c');
alloc.construct(q++, "hi");
//额外参数用来初始化构造的对象
```
**还未构造对象的情况下就使用原始内存是错误的**，必须用`construct`构造对象

**必须对每个构造的元素调用`destroy`来销毁它们**

---

释放内存调用`deallocate`:
```
alloc.deallocate(p, n);
```
传递给`deallocate`的大小参数必须与调用`allocate`分配内存时提供的大小参数具有一样的值。

