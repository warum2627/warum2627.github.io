---
layout: single
title: "stack 구현"
categories: C++
tag: [C++, STL, stack]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
대부분의 내용은 [cplusplus.com]를 참고했습니다.
c++98버전으로 구현합니다.
## 구현 및 설명
전과 마찬가지로 std::와 구분을 하기 위해 namespace를 ft로 진행합니다.
  
## member_type
stack은 기본적으로 컨테이너가 필요합니다. (기본값은 std::deque<T>)
  
먼저 각각의 맴버 타입을 typedef해줍니다.

```c++
namespace ft {
template <class T, class Container = ft::vector<T> >
class stack {
 public:
  typedef T value_type;
  typedef Container container_type;
  typedef typename container_type::size_type size_type;
```
### 맴버 변수
protected 타입으로 최하단에 선언 했으나 이해를 돕기 위해 생성자 전에 설명하겠습니다.
* container_ : 탬플릿 매개변수로 받은 Container 기본은 deque지만, 앞서 구현한 vector를 기본값으로 설정합니다.
```c++
 protected:
  container_type container_;
```
-> stack을 상속받아 사용하는 경우를 대비해 protected로 선언합니다.

### 생성자, 소멸자
생성자는 default 혹은 컨테이너를 매개변수로 받아 생성할 수 있습니다.

* 기본생성자 : 따로 작성하지 않습니다.
* 변환 생성자 : 인자로 받은 컨테이너로 stack을 생성합니다.  
* 대입연산, 복사 생성, 소멸자 등은 기본 기능 입니다.
```c++
   explicit stack(const container_type& ct = container_type()) : container_(ct) {}
  stack(const stack& x) : container_(x.container_) {}
  stack &operator=(const stack &other) {
    if (*this != other) {
      this->container_ = other.container_;
    }
    return (*this);
  }
  ~stack() {}
```

### 맴버 함수
* 각 맴버 함수는 맴버 변수인 container_를 이용해 호출할 수 있도록 합니다.
```c++
  bool empty() const { return (container_.empty()); }
  size_type size() const { return (container_.size()); }
  value_type& top() { return (container_.back()); }
  const value_type& top() const { return (container_.back()); }
  void push (const value_type& val) { container_.push_back(val); }
  void pop () { container_.pop_back(); }
```

### 관계연산자
stack과 stack을 비교하기 위한 연산자 정의 입니다.  
아래의 연산자 정의는 클래스 내부에 있기 때문에 클래스 탬플릿 매개변수와 이름이 같으면 오류가 발생합니다.  
stack는 비교를 도와줄 함수나 iterator가 없기 때문에 protected 변수인 constainer_을 직접 비교 해야 하며, 그로 인해 friend 키워드를 사용합니다.  
(지금까지 구현한 다른 컨테이너는 iterator로 비교 가능)  
유지 보수를 위해 클래스 내부에는 아래와 같이 기본적인 비교 연산자를 선언만 합니다.

```c++
  template <class T1, class C1>
  friend bool operator==(const stack<T1,C1>& lhs, const stack<T1,C1>& rhs);
  template <class T1, class C1>
  friend bool operator<(const stack<T1,C1>& lhs, const stack<T1,C1>& rhs);
```
클래스 내부의 연산자 선언을 정의 합니다.
두 연산자를 이용해 나머지 연산자도 정의 합나디.  

```c++
template <class T, class Container>
bool operator== (const stack<T,Container>& lhs, const stack<T,Container>& rhs) {
  return (lhs.container_ == rhs.container_);
}
template <class T, class Container>
bool operator!= (const stack<T,Container>& lhs, const stack<T,Container>& rhs){
  return !(lhs == rhs);
}
template <class T, class Container>
bool operator<  (const stack<T,Container>& lhs, const stack<T,Container>& rhs) {
  return (lhs.container_ < rhs.container_);
}
template <class T, class Container>
bool operator<= (const stack<T,Container>& lhs, const stack<T,Container>& rhs){
  return !(rhs < lhs);
}
template <class T, class Container>
bool operator>  (const stack<T,Container>& lhs, const stack<T,Container>& rhs){
  return (rhs < lhs);
}
template <class T, class Container>
bool operator>= (const stack<T,Container>& lhs, const stack<T,Container>& rhs){
  return !(lhs < rhs);
}
```

## 결과
42 과제로 진행한 내용이며, 과제 요구사항에 맞게 잘 작동함을 확인했습니다.
개인적으로 테스트한 내용도 있지만, 좋은 테스터가 있어 테스트 결과를 공유합니다.
```shell
# ****************************************************************************** #
#                                                                                #
#                                :::   :::   :::                                 #
#                              :+:+: :+:+:  :+: :+:                              #
#                            +:+ +:+:+ +:+ +:+                                   #
#                           +#+  +:+  +#+ +#+ +#+                                #
#                          +#+       +#+ +#+ #+#                                 #
#                         #+#       #+# #+# #+#                                  #
#                        ###       ### ### ###  containers_test                  #
#                                                                                #
# ****************************************************************************** #
                                   stack
stack/default.cpp                  : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
stack/default_copy.cpp             : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
stack/list_copy.cpp                : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
stack/relational_ope.cpp           : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
stack/relational_ope_list.cpp      : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
```
위의 테스터기는 [marc님의 containers_test] 입니다.

<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>

[cplusplus.com]: http://www.cplusplus.com/reference/stack/stack/?kw=stack
[marc님의 containers_test]: https://github.com/mli42/containers_test