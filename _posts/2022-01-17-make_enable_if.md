---
layout: single
title: "enable_if, is_integral"
categories: C++
tag: [C++, STL, vector]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
백터를 구현하는 중 원하지 않는 탬플릿 함수 호출을 피하기 위해 enable_if, is_integral를 구현합니다.
## 구현 및 설명
전과 마찬가지로 std::와 구분을 하기 위해 namespace를 ft로 진행합니다.
  
### enable_if
enable_if의 작동원리는 간단합니다.
  
먼저 탬플릿 매개변수로 bool타입의 Cond와 기본값은 void로 세팅되어 있는 T를 받는 enable_if 빈 구조체를 선언합니다.
  
그리고 이번에는 bool타입은 없고(true로 고정), T만 받는 enable_if구조체를 선언합니다.
  
아래의 구조체는 typedef를 통해 type을 만들어 놓습니다.

```c++
namespace ft {
template<bool Cond, class T = void>
struct enable_if {};

template<class T>
struct enable_if<true, T> {
  typedef T type;
};
```
이제 아래의 백터 생성자를 예를들어 작동원리를 설명하겠습니다.
```c++
 template<class InputIterator>
  vector(InputIterator first, InputIterator last, const allocator_type &alloc = allocator_type(),
         typename ft::enable_if<!ft::is_integral<InputIterator>::value, InputIterator>::type * = 0)
      : allocator_(alloc), size_(0)
```
4번째 인자인 ```typename ft::enable_if<!ft::is_integral<InputIterator>::value, InputIterator>::type * = 0```를 보면 enable_if의 탬플릿 매개변수로 ```!ft::is_integral<InputIterator>::value``` 가 들어오고, 두번재 매개변수로 ```InputIterator```가 들어왔습니다.
  
즉, 첫번째 매개변수 값이 true라면 InputIterator라는 타입의 typedef인 type가 존재하고 그 타의의 주소값에 0을 넣을 수 있기 때문에 위에 함수가 작동합니다.
  
하지만 첫번째 매개변수 값이 false라면 빈 구조체 이기 때문에 typedef된 type이 없음으로 위의 코드가 작동할 수 없게 됩니다.

컴파일러가 vector(5, 10)을 해석하면서 둘다 정수 타입이기 때문에 위에 탬플릿 함수로 진입을 할 경우  ```is_integral<int>::value```는 true가 되고 `!`연산으로 인해 false가 됩니다.
  
그렇다면 ```enable_if<false, int>```가 되기 때문에 빈 구조체로 인식하고 type라는 타입이 없기 때문에 위의 range생성자가 아닌 fill생성자로 작동 됩니다.

### is_integral
위에서 사용한 is_integral을 구현하려고 합니다.

먼저 인자로 받는 탬플릿에 대한 정보를 담는 integral_constant구조체를 만들어 줍니다.

```c++
namespace ft {
// 아래의 주석 부분처럼 c++11버전이상에서 사용되는것으로 판단.
template<class T, T v>
struct integral_constant {
  static const T value = v;
  typedef T value_type;
  typedef integral_constant<T, v> type;
//    const operator value_type () { return v; }
};
```
위의 구조체에 탬플릿 매개변수 T를 bool로 고정하고 그의 value를 초기화 할 v에 true와 false를 넣어
  
두 버전으로 typedef합니다.
```c++
typedef integral_constant<bool, true> true_type;

typedef integral_constant<bool, false> false_type;
```
위의 두 타입의 구조체를 각각 상속하는 구조체를 선언하는데,
  
탬플릿 매개변수 T를 받는 구조체를 false_type의 구조체를 상속받아 먼저 선언합니다.
  
```c++
template<class T>
struct is_integral : public false_type {};
```
그리고 원하는 타입으로 만들어지는 구조체는 true_type구조체를 상속받습니다.
  
즉, 기본으로는 false_type을 상속 받지만, 지정한 타입은 true_type을 상속받습니다.
  
*지정한 타입은 구조체 이름과 동일하게 정수 타입입니다.*

```c++
template<>
struct is_integral<bool> : public true_type {};

template<>
struct is_integral<char> : public true_type {};

template<>
struct is_integral<wchar_t> : public true_type {};

template<>
struct is_integral<signed char> : public true_type {};

template<>
struct is_integral<short int> : public true_type {};

template<>
struct is_integral<int> : public true_type {};

template<>
struct is_integral<long int> : public true_type {};

template<>
struct is_integral<long long int> : public true_type {};

template<>
struct is_integral<unsigned char> : public true_type {};

template<>
struct is_integral<unsigned short int> : public true_type {};

template<>
struct is_integral<unsigned int> : public true_type {};

template<>
struct is_integral<unsigned long int> : public true_type {};

template<>
struct is_integral<unsigned long long int> : public true_type {};

// std::is_integral 확인결과 cosnt도 처리가 되어 있음
template<>
struct is_integral<const bool> : public true_type {};

template<>
struct is_integral<const char> : public true_type {};

template<>
struct is_integral<const wchar_t> : public true_type {};

template<>
struct is_integral<const signed char> : public true_type {};

template<>
struct is_integral<const short int> : public true_type {};

template<>
struct is_integral<const int> : public true_type {};

template<>
struct is_integral<const long int> : public true_type {};

template<>
struct is_integral<const long long int> : public true_type {};

template<>
struct is_integral<const unsigned char> : public true_type {};

template<>
struct is_integral<const unsigned short int> : public true_type {};

template<>
struct is_integral<const unsigned int> : public true_type {};

template<>
struct is_integral<const unsigned long int> : public true_type {};

template<>
struct is_integral<const unsigned long long int> : public true_type {};
}
```
다시 ```!ft::is_integral<InputIterator>::value``` 부분을 살펴보면 InputIterator 는 int 타입이기 떄문에 true_type을 상속 받습니다.
  
true_type을 상속받은 is_integral의 맴버 value에는 true가 들어 있기 때문에 true이며, `!`연산으로 false가 되는 것입니다.

범위 생성자를 호출 하는 경우 
```c++
  int ints[] = {1, 2, 3};
  ft::vector<int> myvector(inst, ints+1);
```
아래의 코드가 작동하는 원리는
```c++
 template<class InputIterator>
  vector(InputIterator first, InputIterator last, const allocator_type &alloc = allocator_type(),
         typename ft::enable_if<!ft::is_integral<InputIterator>::value, InputIterator>::type * = 0)
      : allocator_(alloc), size_(0)
```
컴파일러가 ```vector(inst, ints+1)```를 해석하면서 둘다 같은 타입이기 때문에 위에 탬플릿 함수로 진입을 합니다.
   
InputIterator의 타입은 정수가아닌 주소 타입이며, 그로 인해 ```is_integral<InputIterator>::value```는 false가 되고 `!`연산으로 인해 true가 됩니다.
  
 ```enable_if<ture, int*>```가 되기 때문에 type라는 타입이 있고, 그 타입의 주소에 0을 넣을 수 있기 때문에 위의 생성자로 진입이 가능합니다.

<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>

[포스트]: http://127.0.0.1:4000/c++/vector/
[cplusplus.com]: https://www.cplusplus.com/reference/vector/vector/?kw=vector