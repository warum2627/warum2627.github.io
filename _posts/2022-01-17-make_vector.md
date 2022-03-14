---
layout: single
title: "vector 구현"
categories: C++
tag: [C++, STL, vector]
toc: true
author_profile: false
# sidebar:
#     nav: "docs"
#search: false 검색을 원하지 않는 경우
---
---
백터의 정의와 기능은 전에 작성한 [포스트] 에서 확인 가능 하며, 대부분의 내용은 [cplusplus.com]를 참고했습니다.
c++98버전으로 구현합니다.
## 구현 및 설명
전과 마찬가지로 std::와 구분을 하기 위해 namespace를 ft로 진행합니다.
  
### member_type
백터는 탬플릿 매개변수로 변수와 타입과, 할당자를 받습니다.
  
할당자는 기본 값으로 std::allocator를 사용합니다.
  
먼저 각각의 맴버 타입을 typedef해줍니다.

```c++
namespace ft {
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
  ```
### 맴버 변수
private 타입으로 최하단에 선언 했으나 이해를 돕기 위해 생성자 전에 설명하겠습니다.
* allocator_ : 할당자 클래스 변수
* size_ : 백터의 요소수
* address_ : 백터 내부에 있는 할당된 메모리의 시작주소
* capacity : 백터에 요소를 넣을 수 있는 크기
```c++
 private:
  allocator_type allocator_;
  size_type size_;
  value_type *address_;
  size_type capacity_;
```

### 생성자
생성자는 default, fill, range 생성할 수 있습니다.
  
fill, range의 경우 잘못된 크기(할당이 불가능한)가 들어오면 throw를 해주는데 std::vector와 동일하게 throw를 할 수 있도록 진행합니다.

* 기본생성자 : 모든 맴버 변수의 값을 0으로 초기합니다.(할당자 제외)
* fill 생성자 : val을 n만큼 채워 생성합니다.
* range 생성자 : 코드 아래에 자세히 설명 하겠습니다.
```c++
  // 기본 생성자
  explicit vector(const allocator_type &alloc = allocator_type())
      : allocator_(alloc), size_(0), address_(0), capacity_(0) {
  }
  // fill
  // std::vectot확인결과 length_error exception으로 vector라고만 출력됨 (맞춤)
  explicit vector(size_type n, const value_type &val = value_type(),
                  const allocator_type &alloc = allocator_type()) : allocator_(alloc) {
    if (n > allocator_.max_size() || n < 0)
      throw std::length_error("vecor");
    size_ = n;
    capacity_ = n;
    address_ = allocator_.allocate(n);
    for (size_type i = 0; i < n; i++) {
      address_[i] = val;
    }
  }
  // range
  // std::vectot확인결과 length_error exception으로 vector라고만 출력됨 (맞춤)
  template<class InputIterator>
  vector(InputIterator first, InputIterator last, const allocator_type &alloc = allocator_type(),
         typename ft::enable_if<!ft::is_integral<InputIterator>::value, InputIterator>::type * = 0)
      : allocator_(alloc), size_(0) {
    difference_type n = ft::distance(first, last);
    if (n > static_cast<difference_type> (allocator_.max_size()) || n < 0)
      throw std::length_error("vector");
    address_ = allocator_.allocate(n);
    capacity_ = n;
    while (first != last) {
      push_back(*first);
      first++;
    }
  }
```

#### range 생성자
주소 또는 iterator의 범위를 통해 생성을 합니다.
  
어떤 iterator가 들어올지 모르기 때문에 탬플릿 함수로 대응을 하는데, **여기서 문제가 발생합니다.**
  
만약 vector의 fill생성자를 이용하기 위해 아래와 같이 생성하면
```c++
  ft::vector<int> myvector(5, 10);
```
우리의 의도는 10이라는 정수를 5개 채운 백터를 만드는 것인데
```c++
template<class InputIterator>
  vector(InputIterator first, InputIterator last, xxxxx)
```
위의 탬플릿 함수에서 InputIterator를 int로 인식해 vector(int 5, int 10, xxxx)로 range생성자가 실행됩니다.
  
이 문제를 해결하기 위해 4번째 매개변수로 enable_if를 사용합니다.
  
enable_if의 자세한 내용은 [여기]에서 확인 가능합니다.
### 복사생성자, 대입연산자, 소멸자
코드의 가독성을 위해 아래에 구현한 push_back, clear등을 이용해 구현했습니다.
  
소멸자의 경우 deallocate실행 전 clear를 통해 destroy로 각 요소의 소멸자를 호출하기 떄문에 누수 발생이 없습니다.

```c++
  // 복사 생성자
  vector(const vector &x) : size_(0), capacity_(x.size_) {
    address_ = allocator_.allocate(capacity_);
    for (size_type i = 0; i < x.size(); i++)
      push_back(x[i]);
  }
  // 대입연산자
  vector &operator=(vector const &other) {
    if (this != &other) {
      // clear()에서  destroy실행 함
      clear();
      allocator_.deallocate(address_, capacity_);
      capacity_ = other.capacity_;
      address_ = allocator_.allocate(capacity_);
      for (size_type i = 0; i < other.size_; i++)
        push_back(other[i]);
    }
    return *this;
  }
  ~vector() {
    if (capacity_ > 0) {
      clear();
      allocator_.deallocate(address_, capacity_);
    }
  }
```
### 맴버 함수 Iterators:
iterator의 시작과 끝을 반환하는 함수입니다.
  
address_는 할당된 메모리의 첫 주소값을 가지고 있기 때문에 address_에 size_(요소 수)를 연산하여 리턴합니다.
```c++
  // vector의 시작 주소값 리턴
  iterator begin() { return iterator(address_); }
  // vector의 시작 주소값을 const로 리턴
  const_iterator begin() const { return const_iterator(address_); }
  // vector의 끝 주소값 리턴
  iterator end() { return iterator(address_ + size_); }
  // vector의 끝 주소값을 const로 리턴
  const_iterator end() const { return const_iterator(address_ + size_); }
  // vector의 끝 주소값을 reverse_iterator로 리턴
  reverse_iterator rbegin() { return reverse_iterator(end()); }
  const_reverse_iterator rbegin() const { return reverse_iterator(end()); }
  // vector의 시작 주소값을 reverse_iterator로 리턴
  reverse_iterator rend() { return reverse_iterator(begin()); }
  const_reverse_iterator rend() const { return reverse_iterator(begin()); }
```

### 맴버함수 Capacity:
백터의 용량에 대한 함수 입니다.

```c++
  // vector의 size를 리턴
  size_type size() const { return size_; }
  // vector의 최대 사이즈를 리턴 (allocator함수 이용)
  size_type max_size() const { return allocator_.max_size(); }
  // vector의 새 크기를 지정한다.
  // value_type val이 있고, 현 size보다 큰 경우 val(val이 없으면 0으로)로 push_back
  // 만약 size보다 n이 작으면 pop_back
  void resize(size_type n, value_type val = value_type(0)) {
    if (n < size_) {
      while (size_ > n)
        pop_back();
    } else {
      while (size_ < n)
        push_back(val);
    }
  }
  // vector의 capacity확인
  size_type capacity() const { return capacity_; }
  // vector가 비어 있는지 확인
  bool empty() const { return (size_ == 0); }
  // vector의 공간을 확보한다.
  // 만약 기존 capacity보다 작은 값을 넣으면 작동하지 않는다.
  // 새로운 n크기의 새로운 백터를 만들어 데이터를 복사해준다.
  // exception은 allocator에서 throw함(std::vector참조)
  void reserve(size_type n) {
    if (n > capacity_) {
      value_type *tmp = allocator_.allocate(n);
      for (size_type i = 0; i < size_; i++) {
        allocator_.construct(tmp + i, address_[i]);
        allocator_.destroy(address_ + i);
      }
      if (capacity_ != 0)
        allocator_.deallocate(address_, capacity_);
      address_ = tmp;
      capacity_ = n;
    }
  }
```

### 맴버함수 Element access:
요소 접근과 관련된 함수 및 연산자 오버로딩 입니다.
```c++
  // vector의 배열연산자 오버로딩
  reference operator[](size_type n) { return address_[n]; }
  const_reference operator[](size_type n) const { return address_[n]; }
  // vector에 지정된 위치에 있는 요소에 대한 참조를 반환한다.
  // std::vectot확인결과 length_error exception으로 vector라고만 출력됨 (맞춤)
  reference at(size_type n) {
    if (n < 0 || n >= size_)
      throw std::out_of_range("vector");
    return address_[n];
  }
  const_reference at(size_type n) const {
    if (n < 0 || n >= size_)
      throw std::out_of_range("vector");
    return address_[n];
  }
  // vector의 첫번째 요소의 참조를 반환
  reference front() { return address_[0]; }
  const_reference front() const { return address_[0]; }
  // vector의 마지막 요소에 참조를 반환
  reference back() { return address_[size_ - 1]; }
  const_reference back() const { return address_[size_ - 1]; }
```

### 맴버 함수 Modifiers:
백터의 수정에 관련된 함수 입니다.
#### assign
백터를 지우고 지정된 요소를 빈 백터에 복사합니다.
  
두 버전이 있으며, 범위의 경우 생성자와 동일하게 enable_if, is_integral를 이용해 잘못된 접근을 막습니다.
```c++
  // fill
  void assign(size_type n, const value_type &val) {
    if (n < 0 || n > allocator_.max_size())
      throw std::length_error("vector");
    clear();
    while (size_ != n)
      push_back(val);
  }
  //range
  template<class InputIterator>
  void assign(InputIterator first,
              InputIterator last,
              typename ft::enable_if<!ft::is_integral<InputIterator>::value, InputIterator>::type * = 0) {
    clear();
    while (first != last) {
      push_back(*first);
      first++;
    }
  }
```
#### push_back
vector의 끝에 요소를 추가합니다.
  
비어있는 vector의 경우 1개의 공간을, 이후로 공간이 부족할 때 2배씩 공간을 할당하는 특징이 있습니다.
  
할당자를 이용하기 때문에 할당에 대한 exception은 std::allocator가 throw 합니다.
```c++
  void push_back(value_type val) {
    if (capacity_ == 0)
      reserve(1);
    if (size_ == capacity_)
      reserve(capacity_ * 2);
    allocator_.construct(address_ + size_, val);
    size_++;
  }
```
#### pop_back
vector의 끝의 요소를 삭제 합니다.
```c++
  void pop_back() {
    allocator_.destroy(address_ + size_ - 1);
    size_--;
  }
```

#### insert
vector 의 지정된 위치에 요소 또는 많은 요소를 삽입합니다.
  
하나의 요소, fill, range등의 오버로딩이 있습니다.
  
하나의 요소를 제외한 두 함수의 작동 분기 점이 있는데,
  
현 백터의 할당된 메모리가 있는지, 부족한지, 충분한지에 따라 다르게 작동합니다.

range에 경우 동일하게 enable_if, is_integral를 이용해 잘못된 접근을 막습니다.
```c++
  // single element
  iterator insert(iterator position, const value_type &val) {
    insert(position, 1, val);
    return position;
  }
  // fill
  void insert(iterator position, size_type n, const value_type &val) {
    iterator iter = begin();
    size_type pos = position.base() - begin().base();
    // std::vector는 세폴이 나오는데, 세폴을 만들 방법이 없어 throw로 처리 함
    if (pos > size_ || pos < 0)
      throw std::length_error("vector");
    // 벡터가 비어있는 경우
    if (capacity_ == 0) {
      address_ = allocator_.allocate(n);
      for (size_type i = 0; i < n; i++) {
        allocator_.construct(address_ + i, val);
      }
      size_ = n;
      capacity_ = n;
    }
      // 다시 할당 해야 하는 경우
    else if (size_ + n > capacity_) {
      size_type j = 0;
      if (size_ + n <= capacity_ * 2)
        capacity_ *= 2;
      else
        capacity_ = size_ + n;
      value_type *tmp = allocator_.allocate(capacity_);
      for (size_type i = 0; i < size_; i++) {
        if (i == pos) {
          while (j < n) {
            allocator_.construct(tmp + i + j, val);
            j++;
          }
        }
        allocator_.construct(tmp + i + j, address_[i]);
        allocator_.destroy(address_ + i);
      }
      allocator_.deallocate(address_, capacity_);
      address_ = tmp;
      size_ += j;
    }
      // capacity가 충분한 경우
    else {
      for (size_type i = 1; i < size_ - pos + n; i++) {
        allocator_.construct(address_ + size_ - i + n, address_[size_ - i - 1 + n]);
        allocator_.destroy(address_ + size_ - i - 1 + n);
      }
      for (size_type i = 0; i < n; i++) {
        allocator_.construct(address_ + pos + i, val);
      }
      size_ += n;
    }
  }
  template<class InputIterator>
  void insert(iterator position,
              InputIterator first,
              InputIterator last,
              typename ft::enable_if<!ft::is_integral<InputIterator>::value, InputIterator>::type * = 0) {
    iterator iter = begin();
    size_type pos = position.base() - begin().base();
    size_type count = ft::distance(first, last);
    // std::vector는 세폴이 나오는데, 세폴을 만들 방법이 없어 throw로 처리 함
    if (pos > size_ || pos < 0)
      throw std::length_error("vecor");
    // InputIterator가 잘못된 주소값이 들어오는 경우 std::vector는 그냥 무시함
    if (count > 100000 || count <= 0) {
    }
      // 벡터가 비어있는 경우
    else if (capacity_ == 0) {
      address_ = allocator_.allocate(count);
      for (size_type i = 0; i < count; i++) {
        allocator_.construct(address_ + i, *first);
        first++;
      }
      size_ = count;
      capacity_ = count;
    }
      // 다시 할당 해야 하는 경우
    else if (size_ + count > capacity_) {
      size_type j = 0;
      if (size_ + count <= capacity_ * 2)
        capacity_ *= 2;
      else
        capacity_ = size_ + count;
      value_type *tmp = allocator_.allocate(capacity_);
      for (size_type i = 0; i <= size_; i++) {
        if (i == pos) {
          while (j < count) {
            allocator_.construct(tmp + i + j, *first);
            j++;
            first++;
          }
        }
        allocator_.construct(tmp + i + j, address_[i]);
        allocator_.destroy(address_ + i);
      }
      allocator_.deallocate(address_, capacity_);
      address_ = tmp;
      size_ += j;
    }
      // capacity가 충분한 경우
    else {
      for (size_type i = 1; i <= count; i++) {
        allocator_.construct(address_ + size_ + count - i, address_[size_ - i]);
      }
      size_ += count;
      for (size_type i = 0; i < count; i++) {
        allocator_.destroy(address_ + pos + i);
        allocator_.construct(address_ + pos + i, *first);
        first++;
      }
    }
  }
```
#### erase
백터에 지정된 위치에서 요소 또는 요소 범위를 제거 합니다.
  
받을 수 있는 매개변수는 typedef되어 있는 iterator입니다.
```c++
  iterator erase(iterator position) {
    pointer pos = position.base();
    if (position + 1 == end()) {
      pop_back();
    }
      // std::vector의 경우 세폴, 세폴을 만들이 어려워 throw로 결정
    else if (end() - position <= 0) {
      throw std::length_error("vector");
    } else {
      allocator_.destroy(pos);
      for (int i = 0; i < end() - position; i++) {
        allocator_.construct(pos + i, *(position + i + 1));
        allocator_.destroy(pos + i + 1);
      }
      size_--;
    }
    return position;
  }
  iterator erase(iterator first, iterator last) {
    iterator tmp = first;
    while (first != last) {
      allocator_.destroy(first.base());
      first++;
    }
    for (int i = 0; i < end() - last; i++) {
      allocator_.construct((tmp + i).base(), *(last + i));
      allocator_.destroy(last.base());
    }
    size_ -= ft::distance(tmp, last, typename ft::iterator_traits<iterator>::iterator_category());
    return tmp;
  }
```
위에서 사용된 distance는 두 iterator사이의 요소 수를 구하는 유틸 함수이며, iterator끼리 연산 가능한 random_access_iterator_tag와 이외의 함수를 따로 오버로딩해 구연했습니다.(효율)
```c++
template<class InputIterator>
typename iterator_traits<InputIterator>::difference_type distance(InputIterator first,
                                                                  InputIterator last,
                                                                  random_access_iterator_tag) {
  return (last - first);
}

template<class InputIterator>
typename iterator_traits<InputIterator>::difference_type distance(InputIterator first, InputIterator last) {
  typedef typename iterator_traits<InputIterator>::difference_type difference_type;
  difference_type cnt = 0;
  while (first != last) {
    cnt++;
    first++;
  }
  return (cnt);
}
```

#### swap
두 vector의 요소를 교환합니다.
  
대입연산 등을 이용할 수 있지만, 메모리를 해제 하고 다시 할당하기 때문에 효율을 고려하여 주소와 각 인자의 값만 복사할 수 있도록 구현했습니다.

```c++
  void swap(vector &x) {
    allocator_type tmp_allocator = x.allocator_;
    size_type tmp_size = x.size_;
    value_type *tmp_address = x.address_;
    size_type tmp_capacity = x.capacity_;

    x.allocator_ = this->allocator_;
    x.size_ = this->size_;
    x.address_ = this->address_;
    x.capacity_ = this->capacity_;

    this->allocator_ = tmp_allocator;
    this->size_ = tmp_size;
    this->address_ = tmp_address;
    this->capacity_ = tmp_capacity;
  }
```

#### clear
vector의 요소를 제거 합니다.
```c++
  void clear() {
    for (size_type i = 0; i < size_; i++) {
      allocator_.destroy(address_ + i);
    }
    size_ = 0;
  }
```

### 맴버 함수 Allocator:
allocator를 반환합니다.
```c++
  allocator_type get_allocator() const { return allocator_; }
```

### 맴버가 아닌 함수
#### relational operators (vector)
백터의 비교 연산자에 자세한 내용은 [std::relational operators (vector)]에서 확인 가능합니다.
  
lhs, rhs는 left- and right-hand side의 약자로 연산자의 왼쪽과 오른쪽을 뜻합니다.
  
`==`의 경우 먼저 크기를 비교하고, 일치하면 요소를 순차적으로 비교하고, 첫 번째 불일치에서 중지 합니다.
  
위의 연산자를 정의하면 `!`를 이용해 `!=`를 구현할 수 있습니다.
```c++
template<class T, class Alloc>
bool operator==(const ft::vector<T, Alloc> &lhs, const ft::vector<T, Alloc> &rhs) {
  return (lhs.size() == rhs.size() && ft::equal(lhs.begin(), lhs.end(), rhs.begin()));
}

template<class T, class Alloc>
bool operator!=(const ft::vector<T, Alloc> &lhs, const ft::vector<T, Alloc> &rhs) {
  return (!(lhs == rhs));
}
```
비교의 경우 lexicographical_compare 알고리즘을 사용합니다.
아래 처럼 `<`를 하나 정의 하면 나머지 비교 연산은 논리로 가능합니다.
* a < b 는 a < b
* a < b 의 반대는 a >= b
* a > b는 b < a
* a > b의 반대는 a <= b 는 !(a > b) 는 !(b < a)
  
```c++
// a < b 는 a < b
template<class T, class Alloc>
bool operator<(const ft::vector<T, Alloc> &lhs, const ft::vector<T, Alloc> &rhs) {
  return (ft::lexicographical_compare(lhs.begin(), lhs.end(), rhs.begin(), rhs.end()));
}

// a < b 의 반대는 a >= b
template<class T, class Alloc>
bool operator>=(const ft::vector<T, Alloc> &lhs, const ft::vector<T, Alloc> &rhs) {
  return (!(lhs < rhs));
}

// a > b는 b < a
template<class T, class Alloc>
bool operator>(const ft::vector<T, Alloc> &lhs, const ft::vector<T, Alloc> &rhs) {
  return (rhs < lhs);
}

// a > b의 반대는 a <= b 는 !(a > b) 는 !(b < a)
template<class T, class Alloc>
bool operator<=(const ft::vector<T, Alloc> &lhs, const ft::vector<T, Alloc> &rhs) {
  return (!(rhs < lhs));
}
```
#### 비교 연산에 사용된 유틸함수
* equal은 요소들이 같은 지 확인하는 함수이며, 같음을 정의하는 조건을 넣을 수 있습니다.
* lexicographical_compare는 요소들의 비교를 해주는 함수이며, 기본적으로는 두번째 탬플릿 매개변수가 큰 경우 true를 리턴합니다.(만약 두 요소가 같다면 false)
   
```c++
template<class InputIterator1, class InputIterator2>
bool equal(InputIterator1 first1, InputIterator1 last1, InputIterator2 first2) {
  while (first1 != last1) {
    if (!(*first1 == *first2))
      return false;
    ++first1;
    ++first2;
  }
  return true;
}

// 특정 조건을 넣어서 같은지 확인(pred)
template<class InputIterator1, class InputIterator2, class BinaryPredicate>
bool equal(InputIterator1 first1, InputIterator1 last1, InputIterator2 first2, BinaryPredicate pred) {
  while (first1 != last1) {
    if (!pred(*first1, *first2))
      return false;
    ++first1;
    ++first2;
  }
  return true;
}

// 두 iter의 집합의 크기 비교 | 왼쪽 값이 작으면 true | 왼쪽 데이터 수가 적어도 ture
template<class InputIterator1, class InputIterator2>
bool lexicographical_compare(InputIterator1 first1, InputIterator1 last1,
                             InputIterator2 first2, InputIterator2 last2) {
  while (first1 != last1) {
    if (first2 == last2 || *first2 < *first1)
      return false;
    else if (*first1 < *first2)
      return true;
    ++first1;
    ++first2;
  }
  return (first2 != last2);
}

// comp라는 비교 조건을 넣어서 함수 실행
template<class InputIterator1, class InputIterator2, class Compare>
bool lexicographical_compare(InputIterator1 first1, InputIterator1 last1,
                             InputIterator2 first2, InputIterator2 last2, Compare comp) {
  while (first1 != last1) {
    if (first2 == last2 || comp(*first2, first1))
      return false;
    else if (comp(*first1, *first2))
      return true;
    ++first1;
    ++first2;
  }
  return (first2 != last2);
}
```


#### swap
동일한 유형의 백터를 서로 교환합니다.
```c++
template<class T, class Alloc>
void swap(ft::vector<T, Alloc> &x, ft::vector<T, Alloc> &y) {
  x.swap(y);
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
                                  vector
vector/assign.cpp                  : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/at.cpp                      : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/at_const.cpp                : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/bidirect_it.cpp             : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/copy_construct.cpp          : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/erase.cpp                   : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/insert.cpp                  : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/insert2.cpp                 : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/ite.cpp                     : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/ite_arrow.cpp               : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/ite_eq_ope.cpp              : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/ite_n0.cpp                  : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [N]
vector/ite_n00.cpp                 : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [N]
vector/ite_n1.cpp                  : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [N]
vector/push_pop.cpp                : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/relational_ope.cpp          : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/rev_ite_construct.cpp       : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/rite.cpp                    : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/rite2.cpp                   : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/rite_arrow.cpp              : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/rite_eq_ope.cpp             : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/size.cpp                    : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
vector/swap.cpp                    : COMPILE: ✅ | RET: ✅ | OUT: ✅ | STD: [Y]
```
위의 테스터기는 [marc님의 containers_test] 입니다.

<div class="notice--primary">
<h4>:interrobang: 본 작성자는 개발자 지망생으로 다소 부족한 부분이 있을 수 있습니다.</h4>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;잘못된 부분이 있다면 이메일 혹은 댓글로 지적 부탁드립니다.:+1:
</div>

[포스트]: http://127.0.0.1:4000/c++/vector/
[cplusplus.com]: https://www.cplusplus.com/reference/vector/vector/?kw=vector
[여기]: https://warum2627.github.io/c++/make_enable_if/
[std::relational operators (vector)]: https://www.cplusplus.com/reference/vector/vector/operators/
[marc님의 containers_test]: https://github.com/mli42/containers_test