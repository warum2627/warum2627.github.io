---
layout: single
title: "std::allocator"
categories: C++
tag: [C++, STL, allocator]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
vector를 생성할 때 기본값으로 설정 되어 있는 std::allocator를 알아보려고 합니다.  
## Allocator란?
```c++
template <class T> class allocator;
```
할당자는 표준 라이브러리의 일부이며, 특히 STL 컨테이너에서 사용할 메모리 모델을 정의하는 클래스 입니다.
(c++에서 일반적인 할당 및 해제는 `new` `delete`연산자를 사용합니다.)
  
### 왜 allocator를 사용하는가?
---
`new` 연산자를 사용하면, 메모리 할당과 동시에 초기화를 진행 합니다.
  
예를 들어 **Cat *c = new Cat();** 와 같은 경우에도 기본 생성자가 호출 됩니다.
  
하지만 `allocator`는 맴버 함수를 통해 단계적으로 진행할 수 있어 세밀한 컨드롤을 할 수 있습니다.

특히 `STL 컨테이너`에서 사용되는 이유는 `allocator`도 탬플릿 클래스로 유연하게 사용할 수 있는 장점이 있습니다.

## Member types
    
|member|definition in allocator|represents|
|:--|:--|:--|
|value_type|T|Element type|
|pointer|T*|Pointer to element|
|reference|T&|Reference to element|
|const_pointer|const T*|Pointer to constant element|
|const_reference|const T&|Reference to constant element|
|size_type|size_t|Quantities of elements|
|difference_type|ptrdiff_t|Difference between two pointers|
|rebind\<Type>|member class|Its member type other is the equivalent allocator type to allocate elements of type Type|
  
백터와 연계해서 생각해보면 `vector<int>` 라고 가정을 했을때
* `vector<int>::reference` 는 `int&`
* `vector<int>::pointer` 는 `int*`
* `vector<int>::size_type` 은 `size_t`
   
인 것을 알 수 있습니다.
  
만약 int 대신 char나 std::string 등이 온다면 유연하게 적용될 수 있습니다.

## Member functions
vector를 구현하면서 사용하게될 함수를 중점으로 알아보겠습니다.
### allocate
```c++
pointer allocate (size_type n, allocator<void>::const_pointer hint=0);
```
`n`은 할당할 크기 입니다. `* sizeof(value_type)`
  
만약 `allocator<int> alloc`으로 `alloc.allocate(5)`을 진행하면 5 * n 만큼의 크기를 할당합니다.
```c++
#include <iostream>
#include <memory>

int main(void) {
  std::allocator<int> ialloc;
  std::allocator<int>::pointer ip;

  ip = ialloc.allocate(5);
  for (int i = 0; i < 5; i++)
    ip[i] = i;

  for (int i = 0; i < 5; i++)
    std::cout << ip[i] << " ";
  std::cout << std::endl;
}
```
위와 같이 사용할 수 있으며 해제 하지 않는 경우
```shell
leaks Report Version: 4.0
Process 4928: 172 nodes malloced for 12 KB
Process 4928: 1 leak for 32 total leaked bytes.

    1 (32 bytes) ROOT LEAK: 0x600003a7d120 [32]
    # 16단위로 할당되며 20의 공간이 필요하기 때문에 32
```
와 같이 누수가 발생합니다.
```c++
#include <iostream>
#include <memory>

int main(void) {
  std::allocator<int> ialloc;
  std::allocator<int>::pointer ip;
  try {
    ip = ialloc.allocate(ialloc.max_size() + 1);
  } catch (std::exception &e) {
    std::cout << e.what() << std::endl;
  }
}
```
와 같이 할당에 문제가 발생하면 `exception`으로 처리합니다.
```shell
allocator<T>::allocate(size_t n) 'n' exceeds maximum supported size
```

### deallocate
```c++
void deallocate (pointer p, size_type n);
```
`p`는 `allocator`의 리턴된 주소를 넣어줘야 합니다.
만약 잘못 된 주소를 넣어주면
```c++
#include <iostream>
#include <memory>

int main(void) {
  std::allocator<int> ialloc;
  std::allocator<int>::pointer ip;

  ip = ialloc.allocate(5);
  for (int i = 0; i < 5; i++)
    ip[i] = i;

  for (int i = 0; i < 5; i++)
    std::cout << ip[i] << " ";
  std::cout << std::endl;

  ialloc.deallocate(ip + 1, 1);
}
```
`abort`에러가 발생하며, `exception`은 발생하지 않습니다.

```shell
0 1 2 3 4
a.out(5737,0x106b45600) malloc: *** error for object 0x600000a9d124: pointer being freed was not allocated
a.out(5737,0x106b45600) malloc: *** set a breakpoint in malloc_error_break to debug
[1]    5737 abort      ./a.out
```

n의 경우 allocate의 n과 동일한 값을 넣어줘야 하는데, 0을 넣어도 문제없이 작동하는(?)부분이 있습니다.
~~`C`의 `free()`와 비슷한???~~ 확인이 필요할 것으로 보입니다.
  
42서울 smun님의 답변에 대한 해석  
>c++ 표준 라이브러리의 명명요구사항에 size_t n을 받도록 작성 되어 있기 때문에 std::allocator도 size_t n을 받도록 되어 있음
size_t n을 받으면 이후 리포트나 디버깅 등으로 사용할 수 있음

```c++
#include <iostream>
#include <memory>

int main(void) {
  std::allocator<int> ialloc;
  std::allocator<int>::pointer ip;

  ip = ialloc.allocate(5);
  for (int i = 0; i < 5; i++)
    ip[i] = i;

  for (int i = 0; i < 5; i++)
    std::cout << ip[i] << " ";
  std::cout << std::endl;
    ialloc.deallocate(ip, 0);
}
```
```shell
Process 6273: 171 nodes malloced for 12 KB
Process 6273: 0 leaks for 0 total leaked bytes.
```

### max_size
```c++
size_type max_size() const throw();
```
할당할 수 있는 최대 요소 수를 리턴합니다.

### construct, destroy 
```c++
void construct ( pointer p, const_reference val );
```
`p`의 위치에 요소 `val`을 생성합니다.
  
**이 함수로는 할당을 할 수 없으며, 할당을 하려면 T의 생성자에서 할당을 할 수 있어야 한다.**

```c++
void destroy (pointer p);
```
`p`의 위치에 있는 요소의 `소멸자`를 실행한다.

**만약 할당을 하는 요소를 이용한 뒤 `destroy`없이 `deallocate`를 하면 `누수`가 발생한다.**
아래의 코드는 의도적으로 **누수**를 발생 시켰다.
```c++
#include <iostream>
#include <memory>

class Tmp {
 public:
  int* numptr;

  Tmp(int num) {
    numptr = new int;
    *numptr = num;
  }
  void print_num() {
    std::cout << *(this->numptr) << std::endl;
  }
  ~Tmp() {
    std::cout << *numptr << "소멸자 실행" << std::endl;
    delete numptr;
  }
};

int main(void) {
  std::allocator<Tmp> talloc;
  std::allocator<Tmp>::pointer tp;
  tp = talloc.allocate(5);
  for (int i = 0; i < 5; i++) {
    talloc.construct(tp + i, i);
  }
  for (int i = 0; i < 4; i++) {
    std::cout << "destroy" << std::endl;
    talloc.destroy(tp + i);
  }
  talloc.deallocate(tp, 5);
}
```
```sell
destroy
0소멸자 실행
destroy
1소멸자 실행
destroy
2소멸자 실행
destroy
3소멸자 실행
leaks Report Version: 4.0
Process 11485: 172 nodes malloced for 12 KB
Process 11485: 1 leak for 16 total leaked bytes.

    1 (16 bytes) ROOT LEAK: 0x600001200090 [16]
```
`destroy`를 실행할 때마다 요소인 `Tmp`의 소멸자가 실행되면서 `int *numptr`를 `delete`하는 것을 알 수 있다.
또한, 마지막 요소는 `destroy`를 하지 않고, `deallocate`를 실행하여 `누수`가 발생하였다.

<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>