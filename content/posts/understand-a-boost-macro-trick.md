---
title: "Understand a Boost Macro Trick"
date: 2023-03-02T09:01:35-08:00
draft: true
---

How is `BOOST_OUTCOME_TRY` macro gets expanded.



BOOST_OUTCOME_TRY(foo(bar))
#define BOOST_OUTCOME_TRY(...) BOOST_OUTCOME_TRY_CALL_OVERLOAD(BOOST_OUTCOME_TRY_INVOKE_TRY, __VA_ARGS__)

=> BOOST_OUTCOME_TRY_CALL_OVERLOAD(BOOST_OUTCOME_TRY_INVOKE_TRY, foo(bar))

#define BOOST_OUTCOME_TRY_CALL_OVERLOAD(name, ...)                                                                                                                   \
  BOOST_OUTCOME_TRY_OVERLOAD_GLUE(BOOST_OUTCOME_TRY_OVERLOAD_MACRO(name, BOOST_OUTCOME_TRY_COUNT_ARGS_MAX8(__VA_ARGS__)), (__VA_ARGS__))

=> BOOST_OUTCOME_TRY_OVERLOAD_GLUE(BOOST_OUTCOME_TRY_OVERLOAD_MACRO(BOOST_OUTCOME_TRY_INVOKE_TRY, BOOST_OUTCOME_TRY_COUNT_ARGS_MAX8(foo(bar))), (foo(bar)))

#define BOOST_OUTCOME_TRY_OVERLOAD_GLUE(x, y) x y

=> BOOST_OUTCOME_TRY_OVERLOAD_MACRO(BOOST_OUTCOME_TRY_INVOKE_TRY, BOOST_OUTCOME_TRY_COUNT_ARGS_MAX8(foo(bar))) (foo(bar))


#define BOOST_OUTCOME_TRY_COUNT_ARGS_MAX8(...) BOOST_OUTCOME_TRY_EXPAND_ARGS((__VA_ARGS__, 8, 7, 6, 5, 4, 3, 2, 1, 0))

=> BOOST_OUTCOME_TRY_OVERLOAD_MACRO(BOOST_OUTCOME_TRY_INVOKE_TRY,
BOOST_OUTCOME_TRY_EXPAND_ARGS((foo(bar), 8, 7, 6, 5, 4, 3, 2, 1, 0))
)?(foo(bar))



#define BOOST_OUTCOME_TRY_EXPAND_ARGS(args) BOOST_OUTCOME_TRY_RETURN_ARG_COUNT args
#define BOOST_OUTCOME_TRY_RETURN_ARG_COUNT(_1_, _2_, _3_, _4_, _5_, _6_, _7_, _8_, count, ...) count

So, in our case, we have only 1,

BOOST_OUTCOME_TRY_EXPAND_ARGS((foo(bar), 8, 7, 6, 5, 4, 3, 2, 1, 0))  ==> 1
BOOST_OUTCOME_TRY_EXPAND_ARGS((foo(bar), g, h, 8, 7, 6, 5, 4, 3, 2, 1, 0))  ==> 3

=> BOOST_OUTCOME_TRY_OVERLOAD_MACRO(BOOST_OUTCOME_TRY_INVOKE_TRY,
1)?(foo(bar))

#define BOOST_OUTCOME_TRY_OVERLOAD_MACRO2(name, count) name##count
#define BOOST_OUTCOME_TRY_OVERLOAD_MACRO1(name, count) BOOST_OUTCOME_TRY_OVERLOAD_MACRO2(name, count)
#define BOOST_OUTCOME_TRY_OVERLOAD_MACRO(name, count) BOOST_OUTCOME_TRY_OVERLOAD_MACRO1(name, count)

=> BOOST_OUTCOME_TRY_INVOKE_TRY1(foo(bar))

#define BOOST_OUTCOME_TRY_INVOKE_TRY1(a) BOOST_OUTCOME_TRYV(a)
=>BOOST_OUTCOME_TRYV(foo(bar))

#define BOOST_OUTCOME_TRYV(...) BOOST_OUTCOME_TRYV2_SUCCESS_LIKELY(BOOST_OUTCOME_TRY_UNIQUE_NAME, __VA_ARGS__)
=>BOOST_OUTCOME_TRYV2_SUCCESS_LIKELY(BOOST_OUTCOME_TRY_UNIQUE_NAME, foo(bar))

#define BOOST_OUTCOME_TRY_UNIQUE_NAME BOOST_OUTCOME_TRY_GLUE(_outcome_try_unique_name_temporary, __COUNTER__)
=>BOOST_OUTCOME_TRYV2_SUCCESS_LIKELY(_outcome_try_unique_name_temporary0, foo(bar))


#define BOOST_OUTCOME_TRYV2_SUCCESS_LIKELY(unique, ...)                                                                                                              \
  auto &&unique = (__VA_ARGS__);                                                                                                                               \
  if(BOOST_OUTCOME_TRY_LIKELY(BOOST_OUTCOME_V2_NAMESPACE::try_operation_has_value(unique)))                                                                                \
    ;                                                                                                                                                          \
  else                                                                                                                                                         \
    return BOOST_OUTCOME_V2_NAMESPACE::try_operation_return_as(static_cast<decltype(unique) &&>(unique))

=>
  auto &&_outcome_try_unique_name_temporary0 = (foo(bar));                                                                                                                               \
  if(BOOST_OUTCOME_TRY_LIKELY(BOOST_OUTCOME_V2_NAMESPACE::try_operation_has_value(_outcome_try_unique_name_temporary0)))                                                                                \
    ;                                                                                                                                                          \
  else                                                                                                                                                         \
    return BOOST_OUTCOME_V2_NAMESPACE::try_operation_return_as(static_cast<decltype(_outcome_try_unique_name_temporary0) &&>(_outcome_try_unique_name_temporary0))



```c++
namespace macro_expansion
{
  // BOOST_OUTCOME_TRY(foo(bar));
  auto && (_outcome_try_unique_name_temporary0) = (foo(bar));
  if(!boost::outcome_v2::try_operation_has_value(_outcome_try_unique_name_temporary0))
    return boost::outcome_v2::try_operation_return_as(static_cast<decltype(_outcome_try_unique_name_temporary0) &&>(_outcome_try_unique_name_temporary0));

  // BOOST_OUTCOME_TRY(x, foo(bar));
  auto && (_outcome_try_unique_name_temporary0) = (foo(bar));
  if(!boost::outcome_v2::try_operation_has_value(_outcome_try_unique_name_temporary0))
    return boost::outcome_v2::try_operation_return_as(static_cast<decltype(_outcome_try_unique_name_temporary0) &&>(_outcome_try_unique_name_temporary0));
  auto && (x) = boost::outcome_v2::try_operation_extract_value(static_cast<decltype(_outcome_try_unique_name_temporary0) &&>(_outcome_try_unique_name_temporary0));

  // auto x = BOOST_OUTCOME_TRYX(foo(bar));
  auto x = ({ auto &&res = (foo(bar));
  if(!boost::outcome_v2::try_operation_has_value(res))
    return boost::outcome_v2::try_operation_return_as(static_cast<decltype(res) &&>(res));
  boost::outcome_v2::try_operation_extract_value(static_cast<decltype(res) &&>(res)); });

  // BOOST_OUTCOME_TRY(x, foo(bar), foo2(bar2))
  auto && (_outcome_try_unique_name_temporary0) = (foo(bar), foo2(bar2));
  if(!boost::outcome_v2::try_operation_has_value(_outcome_try_unique_name_temporary0))
  return boost::outcome_v2::try_operation_return_as(static_cast<decltype(_outcome_try_unique_name_temporary0) &&>(_outcome_try_unique_name_temporary0));
  auto && (x) = boost::outcome_v2::try_operation_extract_value(static_cast<decltype(_outcome_try_unique_name_temporary0) &&>(_outcome_try_unique_name_temporary0));
}
```
