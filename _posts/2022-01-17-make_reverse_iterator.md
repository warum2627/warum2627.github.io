---
layout: single
title: "reverse_iterator 구현"
categories: C++
tag: [C++, STL, vector, reverse_iteraotr]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
member_type나 요소등은 기존 iterator와 비슷하기 때문에 바로 구현을 하고자 합니다.
## reverse_iterator
```c++
template <class Iterator> class reverse_iterator;
```
이 클래스는 `bidirectional` or `random-access iterator`의 방향을 바꿉니다.
원래의 iterator의 복사본은 클래스 내부에서 유지되고 그 복사본을 통해 반대로 작동하도록 합니다.
현 상태의 기본 반복자는 base맴버 함수를 호출해 언제든지 얻을 수 있습니다.

## 구현 및 설명
전과 마찬가지로 std::와 구분을 하기 위해 namespace를 ft로 진행합니다.
  
탬플릿 매개변수는 전에 구현했던 iterator, vector_iterator와 다르게 iterator를 받습니다.
  
자세히 풀어보자면
  
전에 구현했던 vector_iterator의 탬플릿 매개 변수는 변수 타입이고, 그 변수 타입의 포인터 타입을 private변수로 선언해 이동했던 방법과 다르게, reverse_iterator는 만약 vector_iterator를 받으면 그 iterator를 private변수로 선언해 이동합니다.
  
그렇기 때문에 member_type은 탬플릿 매개변수로 받은 iterator를 iterator_traits에 넣어 가져옵니다.
```c++
namespace ft {
template<class Iterator>
class reverse_iterator {
 public:
  typedef Iterator iterator_type;
  typedef typename ft::iterator_traits<Iterator>::iterator_category iterator_category;
  typedef typename ft::iterator_traits<Iterator>::value_type value_type;
  typedef typename ft::iterator_traits<Iterator>::difference_type difference_type;
  typedef typename ft::iterator_traits<Iterator>::pointer pointer;
  typedef typename ft::iterator_traits<Iterator>::reference reference;
  ```
이해를 돕기위해 private 변수를 먼저 작성하겠습니다.
```c++
 private:
  iterator_type current_;
```
생성자, 대입연산자, 소멸자 등의 구현 합니다.
생성자에 추가해야 할 부분은 탬플릿 매개변수로 const타입의 iterator가 들어왔을 때 대응을 해야 합니다.

```c++
// 기본 생성자
  reverse_iterator() : current_(0) {};
// 생성자
  explicit reverse_iterator(iterator_type x) : current_(x) {};
  // const_iter로 const_reverse_iterator를 만들기 위함
  template<class const_iter>
  reverse_iterator(reverse_iterator<const_iter> rev_it) : current_(rev_it.base()) {}
// 복사 생성자
  reverse_iterator(const reverse_iterator<Iterator> &copy) : current_(copy.current_) {};
// 대입연산자
  reverse_iterator &operator=(const reverse_iterator &other) {
    if (this != &other) {
      this->current_ = other.current_;
    }
    return *this;
  };
// 소명자
  ~reverse_iterator() {
  }
```
탬플릿 매개변수로 받았던 iterator를 반납하는 getter 맴버 함수도 작성하고,
  
`연산자*``연산자->`도 구현을 합니다.
단, reverse_iterator는 받은 iterator의 전 요소를 반환합니다.
```c++
  iterator_type base() { return current_; }
  reference operator*() const { return *(current_ - 1); }
  pointer operator->() const { return &(operator*()); }
```
std::reverse_iterator의 작동 예
```c++
#include <iostream>
#include <vector>
int main(void) {
  std::vector<int> myvector;

  for (int i = 1; i < 10; i++)
    myvector.push_back(i); // 1, 2, 3, 4, 5, 6, 7, 8, 9

  std::vector<int>::iterator iter = myvector.begin();
  iter++;
  std::vector<int>::reverse_iterator riter(iter);
  std::cout << *iter << std::endl; // 2
  std::cout << *riter << std::endl; // 1
}
```
전위 후위 증감 또는 산술 및 축약형 연산자는 iterator와 반대로 작동하도록 오버로딩 합니다.
```c++
  reverse_iterator &operator++() {
    current_--;
    return *this;
  }
  reverse_iterator operator++(int) {
    reverse_iterator tmp(*this);
    current_--;
    return tmp;
  }
  reverse_iterator &operator--() {
    current_++;
    return *this;
  }
  reverse_iterator operator--(int) {
    reverse_iterator tmp(*this);
    current_++;
    return tmp;
  }
  reverse_iterator operator+(difference_type n) const { return reverse_iterator(current_ - n); }
  reverse_iterator &operator+=(difference_type n) {
    current_ -= n;
    return (*this);
  }
  reverse_iterator operator-(difference_type n) const { return reverse_iterator(current_ + n); }
  difference_type operator-(reverse_iterator const &other) const { return (other.current_ - current_); }
  reverse_iterator &operator-=(difference_type n) {
    current_ += n;
    return (*this);
  }
```
배열 연산자도 지원합니다. (단, 위의 이유와 마찬가지로 전 요소를 반환합니다.)
  
단순 `==``!=`연산은 동일한 결과를 반환하지만, 나머지 연산은 반대로 반환해줍니다.
```c++
  reference operator[](difference_type n) const { return *(current_ - n - 1); }
  bool operator==(reverse_iterator const &other) const { return (this->current_ == other.current_); }
  bool operator!=(reverse_iterator const &other) const { return (this->current_ != other.current_); }
  bool operator<(reverse_iterator const &other) const { return (this->current_ > other.current_); }
  bool operator>(reverse_iterator const &other) const { return (this->current_ < other.current_); }
  bool operator<=(reverse_iterator const &other) const { return (this->current_ >= other.current_); }
  bool operator>=(reverse_iterator const &other) const { return (this->current_ <= other.current_); }
  operator reverse_iterator<const Iterator>() const { return (reverse_iterator<const Iterator>(this->current_)); }
```

전역함수도 vector_iterator와 동일한 이유로 오버로딩을 해줍니다.
  
단, 단순 `==``!=`연산은 동일한 결과를 반환하지만, 나머지 연산은 반대로 반환해줍니다.
```c++
template<class Iterator>
reverse_iterator<Iterator> operator+(typename reverse_iterator<Iterator>::difference_type n,
                                     const reverse_iterator<Iterator> &i) {
  reverse_iterator<Iterator> new_iter(i);
  return (new_iter + n);
}

template<class Iterator>
reverse_iterator<Iterator> operator-(typename reverse_iterator<Iterator>::difference_type n,
                                     const reverse_iterator<Iterator> &i) {
  reverse_iterator<Iterator> new_iter(i);
  return (new_iter - n);
}

// iter != const_iter, iter >= const_iter 등 처리
template<class Iterator, class Const_iterator>
bool operator!=(ft::reverse_iterator<Iterator> lhs, ft::reverse_iterator<Const_iterator> rhs) {
  return (lhs.base() != rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator==(ft::reverse_iterator<Iterator> lhs, ft::reverse_iterator<Const_iterator> rhs) {
  return (lhs.base() == rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator>(ft::reverse_iterator<Iterator> lhs, ft::reverse_iterator<Const_iterator> rhs) {
  return (lhs.base() < rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator>=(ft::reverse_iterator<Iterator> lhs, ft::reverse_iterator<Const_iterator> rhs) {
  return (lhs.base() <= rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator<(ft::reverse_iterator<Iterator> lhs, ft::reverse_iterator<Const_iterator> rhs) {
  return (lhs.base() > rhs.base());
}

template<class Iterator, class Const_iterator>
bool operator<=(ft::reverse_iterator<Iterator> lhs, ft::reverse_iterator<Const_iterator> rhs) {
  return (lhs.base() >= rhs.base());
```


<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>

[전에 작성한 포스트]: https://warum2627.github.io/c++/base_iterator/
[cplusplus.com]: https://www.cplusplus.com/reference/iterator/iterator/