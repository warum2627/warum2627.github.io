---
layout: single
title: "vector 구현에 필요한 vector_iterator 구현하기"
categories: C++
tag: [C++, STL, iterator, vector]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
vector를 구현하기 전 vector에서 사용할 iterator를 구현하고자 합니다.
## Iterator베이스 클래스 및 traits
### std::iterator_traits
iterator의 특성정보를 확인 하기 위한 클래스 입니다.
  
탬플릿 특수화를 통해 여러가지 타입에 따른 특성을 알 수 있습니다.
  
탬플릿 매개변수로 iterator가 아닌, `T*` `const T*`를 받는 부분이 있는데 STL 알고리즘의 함수는 여러가지 변수의 포인터에도 대응하기 때문입니다.
  
단, iterator의 경우 iterator가 가지고 있는 category를 받는 반면에,
  
포인터는 radom_access_tag 의 기능을 모두 사용 가능함으로 radom_access_tag로 category를 지정합니다.
```c++
#ifndef INCLUDE_ITERATOR_HPP_
#define INCLUDE_ITERATOR_HPP_
#include <cstddef> // size_t, ptrdiff_t

namespace ft {

struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag : public input_iterator_tag {};
struct bidirectional_iterator_tag : public forward_iterator_tag {};
struct random_access_iterator_tag : public bidirectional_iterator_tag {};

template<class Iterator>
struct iterator_traits {
  typedef typename Iterator::difference_type difference_type;
  typedef typename Iterator::value_type value_type;
  typedef typename Iterator::pointer pointer;
  typedef typename Iterator::reference reference;
  typedef typename Iterator::iterator_category iterator_category;
};

template<class T>
struct iterator_traits<T *> {
  typedef ptrdiff_t difference_type;
  typedef T value_type;
  typedef T *pointer;
  typedef T &reference;
  typedef random_access_iterator_tag iterator_category;
};

template<class T>
struct iterator_traits<const T *> {
  typedef ptrdiff_t difference_type;
  typedef T value_type;
  typedef T *pointer;
  typedef T &reference;
  typedef random_access_iterator_tag iterator_category;
};

template<class Category, class T, class Distance = ptrdiff_t,
    class Pointer = T *, class Reference = T &>
struct iterator {
  typedef T value_type;
  typedef Distance difference_type;
  typedef Pointer pointer;
  typedef Reference reference;
  typedef Category iterator_category;
};
}
```
### iterator
  
[전에 작성한 포스트]의 내용대로 구현했습니다.
위와 관련된 내용는 [cplusplus.com]에서 자세히 확인 가능합니다.

## vector_iterator
코드를 작성하기 전 std::와 차이를 주기 위해 namespace를 ft로 합니다.
  
vector_iterator는 위에서 만든 iterator클래스를 상속 받으며, 컨테이너의 특성상 random_access_iterator_tag로 초기화 합니다.

iterator는 결국 포인터의 연산이기 때문에 탬플릿으로 받은 변수의 포인터 타입으로 current_value를 private로 선언합니다.

각각의 Member types를 typedef해주고, 생성자, 대입연산자, 소멸자 등을 구현합니다.

```c++
#include "iterator.hpp"

namespace ft {
template<class Iterator>
class vector_iterator
    : public iterator<ft::random_access_iterator_tag, Iterator> {
 public:
  typedef Iterator iterator_type;
  typedef typename ft::iterator<random_access_iterator_tag, iterator_type>::iterator_category iterator_category;
  typedef typename ft::iterator<random_access_iterator_tag, iterator_type>::value_type value_type;
  typedef typename ft::iterator<random_access_iterator_tag, iterator_type>::difference_type difference_type;
  typedef typename ft::iterator<random_access_iterator_tag, iterator_type>::pointer pointer;
  typedef typename ft::iterator<random_access_iterator_tag, iterator_type>::reference reference;

  // 기본 생성자
  vector_iterator() : current_value(0) {};
  // type의 주소값을 받는 경우의 생성자
  explicit vector_iterator(iterator_type *x) : current_value(x) {};
  // 복사 생성자
  vector_iterator(const vector_iterator<Iterator> &copy) {
    this->current_value = copy.current_value;
  };
  // 대입연산자
  vector_iterator &operator=(const vector_iterator &other) {
    if (this != &other) {
      this->current_value = other.current_value;
    }
    return *this;
  };
  // 소멸자
  virtual ~vector_iterator() {
  }
```
```c++
   private:
  iterator_type *current_value;
  // 실제로는 최하단에 있지만, 이해를 돕기위에 추가 작성
```
private로 선언한 current_value의 getter함수를 만들어 줍니다.
  
iterator의 `*연산`은 `*current_value`로 오버로딩하고 `->연산`은 `*연산의` `&`을 오버로딩 합니다.

```c++
  pointer base() { return current_value; }
  reference operator*() const { return *current_value; }
  pointer operator->() const { return &(operator*()); }
```
전위 후위 증감 감소 등을 모두 지원하기 때문에 연산자 오버로딩을 합니다.
```c++
  // 전위 증감
  vector_iterator &operator++() {
    current_value++;
    return *this;
  }
  // 후위 증감
  vector_iterator operator++(int) {
    vector_iterator tmp(*this);
    current_value++;
    return tmp;
  }
  // 전위 감소
  vector_iterator &operator--() {
    current_value--;
    return *this;
  }
  // 후위 감소
  vector_iterator operator--(int) {
    vector_iterator tmp(*this);
    current_value--;
    return tmp;
  }
```
산술 및 축양형 연산자도 지원하기 때문에 오버로딩 합니다.
  
단, 알고리즘 내부의 함수에서 random_access_tag iterator의 경우 iterator끼리의 `-연산`도 이용하기 때문에 `-연산`도 오버로딩 합니다.

```c++
  vector_iterator operator+(difference_type n) const { return vector_iterator(current_value + n); }
  vector_iterator &operator+=(difference_type n) {
    current_value += n;
    return (*this);
  }
  vector_iterator operator-(difference_type n) const { return vector_iterator(current_value - n); }
  difference_type operator-(vector_iterator const &other) const { return (current_value - other.current_value); }
  vector_iterator &operator-=(difference_type n) {
    current_value -= n;
    return (*this);
  }
```
vector의 iterator는 배열방식의 연산자도 지원합니다.

```c++
  reference operator[](difference_type n) const { return *(current_value + n); }
```
비교 연산자도 오버로딩 합니다.

```c++
  bool operator==(vector_iterator const &other) const { return (this->current_value == other.current_value); }
  bool operator!=(vector_iterator const &other) const { return (this->current_value != other.current_value); }
  bool operator<(vector_iterator const &other) const { return (this->current_value < other.current_value); }
  bool operator>(vector_iterator const &other) const { return (this->current_value > other.current_value); }
  bool operator<=(vector_iterator const &other) const { return (this->current_value <= other.current_value); }
  bool operator>=(vector_iterator const &other) const { return (this->current_value >= other.current_value); }
```
vector컨테이너의 내부 함수를 구현 하면 iterator를 const로 변경해야 하는 경우가 발생합니다.

```c++
  operator vector_iterator<const Iterator>() const {
    return (vector_iterator<const Iterator>(this->current_value));
```

```c++
 private:
  iterator_type *current_value;
};
```
private로 선언한 변수는 위와 같이 최하단에 있지만 이해를 돕기위에 getter함수 전에 한번 더 작성 했습니다.

### vector_iterator에서 사용하는 전역 함수
vector 컨테이너의 경우 iterator + n 뿐만 아니라 n + iterator도 지원을 합니다.
  
클래스 내부에서 오버로딩을 할 경우 매개 변수를 하나만 받을 수 있기 때문에(클래스 자신을 기준으로 연산) 전역에 선언 합니다.

```c++
template<class Iterator>
vector_iterator<Iterator>
operator+(typename vector_iterator<Iterator>::difference_type n, const vector_iterator<Iterator> &i) {
  vector_iterator<Iterator> new_iter(i);
  return (new_iter + n);
}

template<class Iterator>
vector_iterator<Iterator>
operator-(typename vector_iterator<Iterator>::difference_type n, const vector_iterator<Iterator> &i) {
  vector_iterator<Iterator> new_iter(i);
  return (new_iter - n);
}
```
또한 iterator와 const_iteraotr와의 비교연산도 가능하기 때문에
  
(탬플릿 매개변수를 구분하기 위해 Iterator와 Const_iterator로 작성했으나 순서는 상관 없음 / 서로 다른 iterator라는 의미)
  
위와 동일한 이유로 전역에 선언을 합니다.

```c++
template<class Iterator, class Const_iterator>
bool operator!=(ft::vector_iterator<Iterator> lhs, ft::vector_iterator<Const_iterator> rhs) {
  return (lhs.base() != rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator==(ft::vector_iterator<Iterator> lhs, ft::vector_iterator<Const_iterator> rhs) {
  return (lhs.base() == rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator>(ft::vector_iterator<Iterator> lhs, ft::vector_iterator<Const_iterator> rhs) {
  return (lhs.base() > rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator>=(ft::vector_iterator<Iterator> lhs, ft::vector_iterator<Const_iterator> rhs) {
  return (lhs.base() >= rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator<(ft::vector_iterator<Iterator> lhs, ft::vector_iterator<Const_iterator> rhs) {
  return (lhs.base() < rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator<=(ft::vector_iterator<Iterator> lhs, ft::vector_iterator<Const_iterator> rhs) {
  return (lhs.base() <= rhs.base());
}
```

<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>

[전에 작성한 포스트]: https://warum2627.github.io/c++/base_iterator/
[cplusplus.com]: https://www.cplusplus.com/reference/iterator/iterator/