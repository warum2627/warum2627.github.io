---
layout: single
title: "STL vector의 Member type과 typename"
categories: C++
tag: [C++, STL, Member_type, typename]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---  
## vector의 멤버 타입??
[cplusplus.com]에서 vector를 찾아보면 아래와 같은 표가 있습니다.  
  
|member type|definition|notes|
|:--|:--|:--|
value_type|The first template parameter (T)|	
allocator_typy|The second template parameter (Alloc)|defaults to: allocator<value_type>|
reference|allocator_type::reference|for the default allocator: value_type&|
const_reference|allocator_type::const_reference|for the default allocator: const value_type&|
pointer|allocator_type::pointer|for the default allocator: value_type*|
const_pointer|allocator_type::const_pointer|for the default allocator: const value_type*|
iterator|a random access iterator to value_type|convertible to const_iterator|
const_iterator|a random access iterator to const value_type	|
reverse_iterator|reverse_iterator<iterator>	|
const_reverse_iterator|reverse_iterator<const_iterator>	|
difference_type|a signed integral type, identical to: iterator_traits<iterator>::difference_type|usually the same as ptrdiff_t|
size_type|an unsigned integral type that can represent any non-negative value of difference_type|usually the same as size_t|

### 위와 같은 member type을 정의 하는 이유는 무엇일까요?
---
  
위에 표를 보면 `reference`라는 멤버타입이 있고, 그 타입의 정의를 보면 `allocator_type::reference`로 나와 있는데, 이 정의의 `allocator_type` 또한 멤버 타입 입니다.
  
다시 `allocator_type`의 정의를 보면`The second template parameter (Alloc)`로 되어 있고, 두 번째 탬플릿 매개 변수를 찾아보면 `class Alloc = allocator<T>` 로 되어 있습니다.
  
만약 맴버 타입을 지정하지 않고 사용할 경우 `vector`에 `class T`가 `int라면(vector<int>)` `reference`라는 변수타입의 변수를 선언할 때 마다 `std::allocator<int>::reference tmp; `라고 입력 해야 하고, 멤버타입을 지정한 경우는 `reference_type tmp;`와 같이 사용할 수 있습니다.

제가 찾은 이유는 아래와 같습니다.
  
맴버타입(typedef)을 사용하면 읽는 사람 입장에서 타입이름 그대로
>아! 참조자 tmp구나!
  
를 바로 알 수 있고, 작성하는 사람도
  
>음.. 이 함수는 참조자로 리턴하니까 reference_type으로 해야 하겠다!
  
와 같이 사용할 수 있습니다.
  
또한 두번째 탬플릿 매개변수가 기본값은 std::allocator<T>지만, 변경될 경우 일일이 코드를 수정해야 할 수도 있습니다.

위와 같은 이유로 vector를 직접 구현하면서 작성한 맴버 타입은 아래와 같습니다.
```c++
template<class T, class Allocator = std::allocator<T> >
class vector {
 public:
  // member type
  typedef T value_type;
  typedef Allocator allocator_type;
  typedef typename allocator_type::reference reference;
  typedef typename allocator_type::const_reference const_reference;
  typedef typename allocator_type::pointer pointer;
  typedef typename allocator_type::const_pointer const_pointer;
  typedef ft::vector_iterator<value_type> iterator;
  typedef ft::vector_iterator<const value_type> const_iterator;
  typedef ft::reverse_iterator<iterator> reverse_iterator;
  typedef ft::reverse_iterator<const_iterator> const_reverse_iterator;
  typedef typename ft::iterator_traits<iterator>::difference_type difference_type;
  typedef typename allocator_type::size_type size_type;
  //ft 는 forty two의 약자로 직접구현과 std를 구분하기 위한 namespace입니다.

```
  
## 멤버타입은 알겠는데 위에 typename은 무엇일까?
typename은 두 가지 의미가 있는데 이 중 하나는 중첩의존타입이름 입니다.
  
위의 멤버타입으로 예를 들면
T는 말 그대로 탬플릿 매개**변수** T입니다.
Allocator는 말 그대로 탬플릿 두번째 매개**변수** 입니다.
  
그렇다면 **allocator_type::reference는 타입일까요? 변수 일까요??**

아래의 코드를 보면
```c++
#include <iostream>
#include <string>

class Cat {
 public:
  typedef int value_type;

  value_type age_;
  std::string name_;

  Cat(value_type age, std::string name) : age_(age), name_(name) {}
  void speak(void) {
    std::cout << age_ << "살 " << name_ << "입니다." <<std::endl;
  }
};

class Dog {
 public:
  typedef Cat::value_type value_type;

  static int num;
  value_type age_;
  std::string name_;

  Dog(value_type age, std::string name) : age_(age), name_(name) {}
  void speak(void) {
    std::cout << age_ << "살 " << name_ << "입니다." <<std::endl;
    num++;
    std::cout << num << "마리" << std::endl;
  }
};

template<typename T>
class Animal {
 public:
  typedef typename T::value_type value_type;
  value_type age_;
  std::string name_;
  int num;
  Animal(value_type age, std::string name) : age_(age), name_(name) {
    num = T::num;
  }
  void speak(void) {
    std::cout << age_ << "살 " << name_ << "입니다." <<std::endl;
    num++;
    std::cout << num << "마리" << std::endl;
  }
};

int Dog::num = 0;
int main(void) {
  Cat cat(1, "navi");
  Dog dog(2, "walwal");

  cat.speak();
  dog.speak();

  Animal<Dog> animal2(3, "ho");
  animal2.speak();
}
```
탬플릿 클래스 Animal에 정적 변수의 값을 대입하는 `num = T::num;` 부분과 변수타입을 typedef하는 `typedef typename T::value_type value_type;` 부분이 `T::xxxx`로 동일한 것을 볼 수 있습니다.
  
사람은 코드를 해석하면서
  
>아! 타입이구나
  
를 알 수 있지만, 컴파일러는 변수인지 타입인지 알 수 없다고 합니다.(기본으로는 타입이 아니라고 가정한다고 합니다.)
  
그렇기 때문에 타입이라는 것을 알려줘야 하고, 위의 value_type은 T라는 클래스에 따라 의존 되어 변경 되며, T라는 클래스에 포함되어 있기 때문에 중첩의존타입이름 이라고 합니다.
  
다행인 부분은 컴파일러가 똑똑해서 위와 같은 코드에 typename을 입력 하지 않으면
```shell
tmp.cpp:36:11: error: missing 'typename' prior to dependent type name 'T::value_type'
  typedef T::value_type value_type;
          ^~~~~~~~~~~~~
          typename
1 error generated.
```
와 같이 친절하게 알려줍니다.
  
추가로 Dog 클래스 내부의 `typedef Cat::value_type value_type;`는 typename 키워드가 없는데, 어떤 타입이 들어올지 모르는 탬플릿과 다르게 Cat이 class임을 컴파일러가 알고 있기 때문입니다.
  
<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>

[cplusplus.com]: (https://www.cplusplus.com/reference/vector/vector/?kw=vector)