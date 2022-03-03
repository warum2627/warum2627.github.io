---
layout: single
title: "map 구현에 필요한 map_iterator 구현하기"
categories: C++
tag: [C++, STL, iterator, vector]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
map를 구현하기 전 사용할 iterator를 구현하고자 합니다.
## map에서 사용할 노드의 구성
아래의 노드는 map클래스 내부에 작성되어 있습니다.  
STL과 동일하게 pair로 되어 있는 content를 저장합니다.  
이진탐색트리를 사용하기 위해 left, right의 자녀노드와 부모노드를 가리킬 포인터를 선언합니다.  
저는 균형 이진탐색트리를 구현하기 때문에 균형을 잡기위해 height를 선언합니다.  
```c++
 private:
  struct Node {
    ft::pair<Key, T> content;
    int height;
    Node *parent;
    Node *left;
    Node *right;
  };
```
사실 구현을 도전하면서, vector처럼 iterator를 먼저 정의하기 어려웠습니다.  
map의 private함수로 노드의 생성, 탐색, 삭제 등을 먼저 구현한 뒤 균형을 잡고, 그 노드를 대상으로 iterator를 구현했기 때문입니다.  
(map클래스를 구현 하다가 map_iteraotr를 구현하고 map클래스를 수정하는 등의 반복)  
이 부분은 진행하시는 방법에 따라 차이가 있을 것으로 예상됩니다.  

## map_iterator
코드를 작성하기 전 std::와 차이를 주기 위해 namespace를 ft로 합니다.
  
map_iterator는 전에 만든 iterator클래스를 상속 받으며, 컨테이너의 특성상 bidirectional_iterator_tag로 초기화 합니다.

map은 내부에서 노드를 이용하기 때문에 노드 포인터와 노드의 끝을 알기 위해 last노드 포인터를 인자로 받습니다.
(last->right는 root를 가리키도록 설정)

각각의 Member types를 typedef해주고, 생성자, 대입연산자, 소멸자 등을 구현합니다.

```c++
#include "iterator.hpp"

namespace ft {
template<class Iterator, class Compare, class Value_type>
class map_iterator : ft::iterator<ft::bidirectional_iterator_tag, Iterator> {
 public:
  // typedef
  // iterator_type은 노드
  typedef Iterator iterator_type;
  typedef iterator_type* node_pointer;
  typedef typename ft::iterator<ft::bidirectional_iterator_tag, Value_type>::iterator_category iterator_category;
  typedef Value_type value_type;
  typedef typename ft::iterator<ft::bidirectional_iterator_tag, value_type>::difference_type difference_type;
  typedef typename ft::iterator<ft::bidirectional_iterator_tag, value_type>::pointer pointer;
  typedef typename ft::iterator<ft::bidirectional_iterator_tag, value_type>::reference reference;


 public:
  // construct, ...
  map_iterator() : node_(0), last_(0) {}

  // 구현할 map의 내부 구조는 이진탐색트리로, 한번 할당 된 last가 각 노드의 끝을 가리키고 있기 때문에
  // last노드를 알아야 iterator로 문제 없이 탐색할 수 있다.
  explicit map_iterator(node_pointer node, node_pointer last) : node_(node), last_(last) {}

  map_iterator(const map_iterator &copy) : node_(copy.node_), last_(copy.last_) {}

  map_iterator &operator=(const map_iterator &other) {
    if (this != &other) {
      node_ = other.node_;
      last_ = other.last_;
    }
    return *this;
  }

  ~map_iterator() {}
```
```c++
 private:
  node_pointer node_;
  node_pointer last_;
  // 실제로는 최하단에 있지만, 이해를 돕기위에 추가 작성
```
private로 선언한 노드 포인터의 getter함수를 만들어 줍니다.
  
map_iterator는 vector와 달리 `*연산`은 `노드가 가지고 있는 content의 값을 바로 꺼내 줍니다.` `->연산`은 `content의 주소를 리턴합니다.`

```c++
 node_pointer const base() { return node_; }
  // operator..
  bool operator==(map_iterator const &other) const { return (this->node_ == other.node_); }
  bool operator!=(map_iterator const &other) const { return (this->node_ != other.node_); }
  reference operator*() const { return node_->content; }
  pointer operator->() const { return &(node_->content); }
```

전위 후위 증감 감소 등을 모두 지원하기 때문에 연산자 오버로딩을 합니다.  
이진탐색트리의 특성에 따라 노드의 위치를 변경합니다.  
```
         4
   2           6
1     3     5     7
```
위의 경우에서 만약 현재 노드가 4라면 그다음 숫자는 한번 오른쪽으로 이동한 뒤 제일 왼쪽에 있는 5 노드입니다.  
현재 위치가 1이라면 오른쪽 노드가 없고, 부모노드 의 입장에서 본인이 왼쪽이기 때문에 부모 노드인 2가 그 다음 값입니다.  
현재 위치가 3이라면 자식 노드가 없고, 부모노드 입장에서 본인은 오른쪽 이기 때문에 한번 올라가고 2는 부모입장에서 왼쪽이기 때문에 한번만 더 올라간 뒤 4 노드로 변경 후 종료 됩니다.  
만약 7이라면 계속 부모노드의 오른쪽 이기 때문에 last를 반환합니다.  

```c++
  // 전위 증감
    map_iterator &operator++() {
    if (node_ != NULL) {
      if (node_->right != last_) {
        node_ = node_->right;
        while (node_->left != last_) {
          node_ = node_->left;
        }
      }
      else {
        while (node_->parent != last_ && node_->parent->right == node_)
          node_ = node_->parent;
        node_ = node_->parent;
      }
    }
    return *this;
  }

  map_iterator operator++(int) {
    map_iterator tmp(*this);
    operator++();
    return tmp;
  }
```
```
         4
   2           6
1     3     5     7
```
  
만약 현재 위치가 last라면 전 값은 가장 큰 값인 7으로 옮겨져야 합니다.  
last노드의 오른쪽에는 만들어진 트리의 root(4)를 참조 하고 있으며 그 값을 이용해 가장 오른쪽 값인 7로 이동합니다.  
현재 노드가 4라면 전 숫자는 한번 왼쪽으로 이동한 뒤 제일 오른쪽에 있는 3 노드입니다.  
현재 위치가 7이라면 왼쪽 노드가 없고, 부모노드 의 입장에서 본인이 오른쪽이기 때문에 부모 노드인 6이 그 다음 값입니다.  
현재 위치가 5라면 자식 노드가 없고, 부모노드 입장에서 본인은 왼쪽 이기 때문에 한번 올라가고 6은 부모입장에서 오른쪽이기 때문에 한번만 더 올라간 뒤 4 노드로 변경 후 종료 됩니다.  
만약 1이라면 계속 부모노드의 왼쪽 이기 때문에 last를 반환합니다.  

```c++
  map_iterator &operator--() {
    if (node_ != NULL) {
      if (node_ == last_) {
        node_ = node_->right;
        while (node_->right != last_)
          node_ = node_->right;
      }
      else if (node_->left != last_) {
        node_ = node_->left;
        while (node_->right != last_)
          node_ = node_->right;
      }
      else {
        while (node_->parent != last_ && node_->parent->left == node_)
          node_ = node_->parent;
        node_ = node_->parent;
      }
    }
    return *this;
  }

  map_iterator operator--(int) {
    map_iterator tmp(*this);
    operator--();
    return tmp;
  }
```

map컨테이너의 내부 함수를 구현 하면 iterator를 const로 변경해야 하는 경우가 발생합니다.

```c++
  operator map_iterator<Iterator, Compare, const Value_type>() const {
    return (map_iterator<Iterator, Compare, const Value_type>(this->node_, this->last_));
  }
```

```c++
 private:
  node_pointer node_;
  node_pointer last_;
};
```
private로 선언한 변수는 위와 같이 최하단에 있지만 이해를 돕기위에 getter함수 전에 한번 더 작성 했습니다.


<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>
