# C++ Template Metaprogramming Examples

Most of the examples below use SPINAE to achieve template metaprogramming.

Let's get started from easy to more complex.

## Implement is_void\<T\>
```
template<typename T> struct is_void{
	enum{value = false};
};

template<> struct is_void<void>{
	enum{value = true};
};

template<> struct is_void<const void>{
	enum{value = true};
};
```

## Implement is_same\<T\>
```
template<class T, class U> struct is_same{ // primary template
	enum{value = false};
};
template<class T> struct is_same<T, T>{
	enum{value = true};
};
```
## Check whether a type is in a list
```
template<typename T, typename ...Ts> struct is_one_of;
template<typename T, typename ...Ts> struct is_one_of<T, T, Ts...>{
	enum{value = true};
};

template<typename T> struct is_one_of<T>{
	enum{value = false};
};

template<typename T, typename P, typename ...Ts> struct is_one_of<T, P, Ts...>{
	enum{value = is_one_of<T, Ts...>::value}; // Recursive call.
};
```

## Check whether a list of types is unique
```
template<typename T, typename ...Ts> struct is_unique;
template<typename T, typename ...Ts> struct is_unique<T, T, Ts...>{
	enum{value = false};
};

template<typename T> struct is_unique<T>{
	enum{value = true};
};

template<typename T, typename P, typename ...Ts> struct is_unique<T, P, Ts...>{
	enum{value = is_unique<T, Ts...>::value && is_unique<P, Ts...>::value}; // Recursive call.
};
```

## 3 ways to check whether there is a public method called 'test'

### 1. A NULL paramter could be accepted by a method as a pointer
```
template<typename T> struct has_member_test_v1 {
	private:
	template<typename U>
	static constexpr bool check(...) { return false;}
	template<typename U>
	static constexpr bool check(decltype(&U::test)) {return true;} // 1. A pointer to a function also accepts NULL;
	public:
	enum{value = check<T>(NULL)};
};
```
### 2. Use a redundant default template parameter
```
template<typename T> struct has_member_test_v2 {
	private:
	static constexpr false_type check(...); // Only yield return type, no need to implement.
	template<typename U, typename _ = decltype(&U::test)> // 2. A redundant default template parameter.
	static constexpr true_type check(U &&);
	public:
	enum{value = decltype(check(declval<T>()))::value};
};
```
### 3. Remove checking method from 2.
```
template<typename T, typename _ = void> // 3. A redundant typed template parameter.
struct has_member_test_v3 { enum{value = false}; };

template<typename T> 
struct has_member_test_v3<T, void_t<decltype(&T::test)>> { // 3. The redundant typed template parameter accepts a function pointer.
	enum{value = true};
};
```
## 4 ways to check whether a type is a pointer
### 1. Use function to check object directly
```
struct is_pointer_v1 {
	static constexpr bool check(...){return false;}
	template<typename U> static constexpr bool check(U*) {return true;} // 1. Function check object.
};
```
### 2. Use declval\<T\>() to create the object, and pass it to the method.
```
template<typename T>
struct is_pointer_v2 {
	private:
	static constexpr false_type check(...);
	template<typename U> static constexpr true_type check(U*);
	public:
	enum{value = decltype(check(declval<T>()))::value}; // declval<T>() to create the object, and pass it to the method.
};
```
### 3. Use template specilization for the pointer
```
template<typename T> struct is_pointer_v3{enum{value = false};};
template<typename T> struct is_pointer_v3<T*>{enum{value = true};}; // template specilization.
```
### 4. Try to dereference the type
```
template<typename T, typename = void> struct is_pointer_v4{enum{value = false};};
template<typename T> struct is_pointer_v4<T, void_t<decltype(*declval<T>())>>{enum{value = true};}; // Try to dereference the type, and change the type to void type so that we don't need to supply the parameter.
```

## Implement a Tuple type
```
template<typename T, typename ... Args>
struct Tuple {
	Tuple(T t, Args... args) :value(t), rest(args...){}
	T value;
	Tuple<Args...> rest;
	static constexpr int size() { return 1 + Tuple<Args...>::size(); }
};
template<typename T>
struct Tuple<T> {
	Tuple(T t) :value(t){}
	T value;
	static constexpr int size() { return 1;}
};

template<int N>
struct getter {
	template<typename ... Args>
	static auto& get(Tuple<Args...>& tuple) {
		return getter<N - 1>::get(tuple.rest);
	}
};

template<>
struct getter<0> {a
	template<typename T, typename... Args>
	static T& get(Tuple<T, Args...>& t) {
		return t.value;
	}
};
```

## Test code
```
#include <iostream>
  
int main(int, char **)
{
    std::cout <<"is_void<int>: " << is_void<int>::value << "\n";
    std::cout <<"is_void<void const>: " << is_void<void const>::value << "\n";
    std::cout <<"is_void<const void>: " << is_void<const void>::value << "\n";
    std::cout <<"is_void<void>: " << is_void<void>::value << "\n\n";
    
    std::cout <<"is_same<int,void>: " << is_same<int,void>::value << "\n";
    std::cout <<"is_same<int,int>: " << is_same<int,int>::value << "\n\n";
    
    std::cout <<"is_one_of<void, int, char, double>: " << is_one_of<void, int, char, double>::value << "\n";
    std::cout <<"is_one_of<bool, void, int, char, double, void>: " << is_one_of<bool, void, int, char, double, void>::value << "\n";
    std::cout <<"is_one_of<void, int, char, double, void>: " << is_one_of<void, int, char, double, void>::value << "\n\n";

    std::cout <<"is_unique<void, int, char, double>: " << is_unique<void, int, char, double>::value << "\n";
    std::cout <<"is_unique<bool, void, int, char, double, void>: " << is_unique<bool, void, int, char, double, void>::value << "\n";
    std::cout <<"is_unique<void, int, char, double, void>: " << is_unique<void, int, char, double, void>::value << "\n\n";
    
    struct test_struct{int test(){return 0;}};
    std::cout << "has_member_test_v1<string>: " << has_member_test_v1<string>::value << "\n";
    std::cout << "has_member_test_v1<test_struct>: " << has_member_test_v1<test_struct>::value << "\n";
    std::cout << "has_member_test_v2<string>: " << has_member_test_v2<string>::value << "\n";
    std::cout << "has_member_test_v2<test_struct>: " << has_member_test_v2<test_struct>::value << "\n";
    std::cout << "has_member_test_v3<string>: " << has_member_test_v3<string>::value << "\n";
    std::cout << "has_member_test_v3<test_struct>: " << has_member_test_v3<test_struct>::value << "\n\n";
    
    int p1, *p2;
    std::cout << "is_pointer_v1::check(p1): " << is_pointer_v1::check(p1) << "\n";
    std::cout << "is_pointer_v1::check(p2): " << is_pointer_v1::check(p2) << "\n";
    std::cout << "is_pointer_v2<typeof(p1)>: " << is_pointer_v2<decltype(p1)>::value << "\n";
    std::cout << "is_pointer_v2<typeof(p2)>: " << is_pointer_v2<decltype(p2)>::value << "\n";
    std::cout << "is_pointer_v3<typeof(p1)>: " << is_pointer_v3<decltype(p1)>::value << "\n";
    std::cout << "is_pointer_v3<typeof(p2)>: " << is_pointer_v3<decltype(p2)>::value << "\n";
    std::cout << "is_pointer_v4<typeof(p1)>: " << is_pointer_v4<decltype(p1)>::value << "\n";
    std::cout << "is_pointer_v4<typeof(p2)>: " << is_pointer_v4<decltype(p2)>::value << "\n\n";
    
    Tuple<int, char, double> t(10, 'a', 3.14);
    std::cout << "getter<0>::get(t): " << getter<0>::get(t) << ", getter<1>::get(t): " << getter<1>::get(t) << ", getter<2>::get(t): " << getter<2>::get(t) << "\n";
    return 0;
}
```
## Output:
> is_void<int>: 0
> is_void<void const>: 1
> is_void<const void>: 1
> is_void<void>: 1
> 
> is_same<int,void>: 0
> is_same<int,int>: 1
> 
> is_one_of<void, int, char, double>: 0
> is_one_of<bool, void, int, char, double, void>: 0
> is_one_of<void, int, char, double, void>: 1
> 
> is_unique<void, int, char, double>: 1
> is_unique<bool, void, int, char, double, void>: 0
> is_unique<void, int, char, double, void>: 0
> 
> has_member_test_v1<string>: 0
> has_member_test_v1<test_struct>: 1
> has_member_test_v2<string>: 0
> has_member_test_v2<test_struct>: 1
> has_member_test_v3<string>: 0
> has_member_test_v3<test_struct>: 1
> 
> is_pointer_v1::check(p1): 0
> is_pointer_v1::check(p2): 1
> is_pointer_v2<typeof(p1)>: 0
> is_pointer_v2<typeof(p2)>: 1
> is_pointer_v3<typeof(p1)>: 0
> is_pointer_v3<typeof(p2)>: 1
> is_pointer_v4<typeof(p1)>: 0
> is_pointer_v4<typeof(p2)>: 1
> 
> getter<0>::get(t): 10, getter<1>::get(t): a, getter<2>::get(t): 3.14

## References
  
CppCon 2014: Walter E. Brown "Modern Template Metaprogramming: A Compendium, Part I"
  
https://youtu.be/Am2is2QCvxY
  
CppCon 2014: Walter E. Brown "Modern Template Metaprogramming: A Compendium, Part II"
  
https://youtu.be/a0FliKwcwXE
