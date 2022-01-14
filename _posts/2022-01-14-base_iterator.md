---
layout: single
title: "iterator"
categories: C++
tag: [C++, STL, iterator]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
vector를 구현하기 전 STL라이브러리에서 사용되는 iterator에 대해 알아보려고 합니다.
## Iterator란?
C++만의 개념이 아닌 python, java등에서도 사용합니다.
컨테이너와 일반 알고리즘 사이에서 중개자 역할을 하며, 서로 다른 데이터 구조에서 동일한 방식으로 작동하도록 포인터의 일반화를 구현합니다.

## 서로 다른 데이터구조에서 iterator를 사용하는 의미
STL 컨테이너는 vector처럼 단일 메모리 블럭으로 되어 있는 구조도 있지만, 여러 메모리 블럭에 퍼져있는 구조의 컨테이너도 있습니다.
  
여러 컨테이너의 요소에 접근을 하고 이동을 하려면 주소값을 `++` `--`등으로 이동할 수도 있지만 `curr=curr->next`와 같이 직접적으로 주소를 옮겨야할 수도 있습니다.
  
하지만 iterator를 사용하면 `++`와 같이 동일한 방식으로 이동이 가능하다는 것입니다.
  
즉 iterator도 여러가지 종류가 있고 그 종류에 따라 외부에서는 동일해 보이지만 내부에서는 서로 다른 작동방식을 가지고 있다는 뜻입니다.

## Iterator의 base 클래스
```c++
template <class Category,              // iterator::iterator_category
          class T,                     // iterator::value_type
          class Distance = ptrdiff_t,  // iterator::difference_type
          class Pointer = T*,          // iterator::pointer
          class Reference = T&         // iterator::reference
          > class iterator;
```
반복자 클래스를 파생시키는 데 사용할 수 있는 기본 클래스 탬플릿입니다.
### category
반복자는 아래에 있는 카테고리 태그 중 하나를 반드시 가지고 있어야 합니다.
  
|iterator tag|Category of iterators|
|:--|:--|
|input_iterator_tag|Input Iterato|
|output_iterator_tag|Output Iterator|
|forward_iterator_tag|Forward Iterator|
|bidirectional_iterator_tag|Bidirectional Iterator|
|random_access_iterator_tag|Random-access Iterator|

### iterator tag의 구성
```c++
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag : public input_iterator_tag {};
struct bidirectional_iterator_tag : public forward_iterator_tag {};
struct random_access_iterator_tag : public bidirectional_iterator_tag {};
```
tag마다의 큰 특징은 연산에 차이가 있습니다.
  
간단한 예를 들면 bidirectional_iterator_tag의 경우  `==` `!=` 연산이 가능하지만 `>` `<` `>=` `<=` 등의 비교 연산이 불가능합니다.

하지만 random_access_iterator_tag의 경우 bidirectional_iterator_tag의 `==` `!=` 연산 기능을 포함한 `>` `<` `>=` `<=` 와 같은 연산이 가능합니다.
  
위의 코드를 보면 각각의 tag를 상속관계로 설정함으로써(구조체) tag끼리 상관관계를 이해하기 쉽게 구성한 것을 볼 수 있습니다.
  
STL라이브러리의 함수 중에는 iterator의 연산이 필요한 경우 tag를 확인하는 경우가 있습니다.
  
예를 들면 std::sort 처럼 random_access_iterator_tag가 아닌경우 작동할 수 없는 함수도 있고, 각 함수에서 iterator_tag에 따라 보다 더 효율적인 계산을 할 수 있기 때문입니다.
```c++
#include <iostream>
#include <memory>
#include <map>
int main(void) {
  std::map<int, int> mymap;

  for(int i = 10; i > 0; i--)
    mymap[i] = i;

  std::map<int, int>::iterator map_iter_first = mymap.begin();
  std::map<int, int>::iterator map_iter_last = mymap.end();

  while (map_iter_first != map_iter_last) {
    std::cout << map_iter_first->first << std::endl;
    std::cout << map_iter_first->second << std::endl;
    map_iter_first++;
  }
  map_iter_first = mymap.begin();
  std::sort(map_iter_first, map_iter_last);
}
```
위의 코드를 컴파일 하면 엄청나게 많은 오류를 찾아볼 수 있습니다.
대표적으로 std::sort 내부에서 요소의 수를 구하기 위한 코드가 있는데
  
```difference_type __len = __last - __first;```
  
random_access_iterator_tag의 iterator는 `+` `-` 등의 연산이 가능하지만, bidirectional_iterator_tag의 iterator는 불가능 하기 때문에 위의 계산을 할 수 없습니다.

위의 내용을 종합 해보면
각 컨테이너는 데이터의 구조에 따라 iterator를 다르게 구성하고 있고,
그 확인은 iteraotr클래스에 있는 category를 통해 하고 있다는 뜻입니다.

### T
반복자가 가리키는 요소의 유형
### Distance
두 iterator간의 차이를 나타내는 유형이며 기본 타입은 ptrdiff_t 입니다.
### Pointer
말 그대로 T* T의 포인터 타입입니다.
### Reference
T& T의 참조 타입입니다.
### Member types
  
|member type|definition|notes|
|:--|:--|:--|
|iterator_category|the first template parameter (Category)||	
|value_type|the second template parameter (T)||	
|difference_type|the third template parameter (Distance)|defaults to: ptrdiff_t|
|pointer|the fourth template parameter (Pointer)|defaults to: T*|
|reference	the fifth template parameter (Reference)|defaults to: T&|

## 결론
STL의 vector를 구현하려면 vector의 요소에 접근할 수 있는 iterator가 필요하고 iterator는 위의 기본적인 클래스를 상속 받아야 합니다.
```c++
template<class Iterator>
class vector_iterator
    : public iterator<ft::random_access_iterator_tag, Iterator> {
  //ft 는 forty two의 약자로 직접구현과 std를 구분하기 위한 namespace입니다.
```
vector는 random_access가 가능하기 때문에 random_access_tag만 지정을 해주고 나머지는 기본값으로 사용할 계획입니다.


<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>