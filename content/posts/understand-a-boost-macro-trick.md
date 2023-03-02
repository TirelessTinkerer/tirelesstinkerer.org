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



[Link](
https://godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGIM1ykrgAyeAyYAHI%2BAEaYxCCSZqQADqgKhE4MHt6%2B/oGp6Y4CoeFRLLHxibaY9kUMQgRMxATZPn4B1bWZDU0EJZExcQlJCo3Nrbkdo739ZRXDAJS2qF7EyOwc5gDMYcjeWADUJltu0ahpBAD0KwRobJcExACeAHQIycnH2CYaAILbu32mCOJ1GxDCwC%2BP3%2BZh2DD2XkOxzcTjBmFYUL%2BAPhQJBp3Oo2uXluqHumFUyTieDYgjEl2mBC8CgA%2Bjc7pg3h9Mf8/owfAdsMRiCQPFhsAwfAojgB2Kx/A4Kg4AMV%2BAElgiCACIHQLQxUHIRyNxubBCIRag4aUjQkzSzXHOU835grwOAVCkUYYG2x36wXC4iinASlhSzDIZkOvWK0boEAgMEQg6h4CRra%2BxX%2Bz1YCALC1YKhMLy0AhR%2BWZj2Br0QLNVsUhqVoLCkA6x%2BOJozJzAKBRMYCYPMgA7h5kQJsDlsp0dsHt9gcyqx2sv/Jfpm1/AiYFjJAybvF7Jg9g4AFS%2B%2BvPF8vV%2BvN9vd/vt%2BhTKTACVu8WCBaLAB5b9CY/Mt%2BcjHm434ALLYMyABqZjMhEvwQUIyi/Ca8bkpS4I0o0tDto0jIssQ74lsix4trWQbWuWD7UTRtGXj%2Bf4AUBIHgZBMFwQhprIahICpLQeDIE88absQLBhEwm5QmuWJ/M%2BnaqoIb4KB%2BFpKR%2ByJhKWWzfNJaqKURn5UBAZyeAcVAGMAeY%2BtGCqaQchHKSWFpcBoGjLvqeBUAc1AWVZso2eehGMsQDD2QZ7mKra2o1EoC4BfqQWrKF5HViq6otuYZjYMkAnKkwdCrJgmULBFCpRTaS4yXpBBqU5wDGectBmb5cVUQcdkOSpxzai5bm6R5Xk%2BX2fkZheiUhWFjlaaNMrRbQsXWW1CWYMFyWVkGEBpcEGVmFlOXIHlBWEcVpWzRV9pVXZLD5Qwuatb8CpFkQByqF%2Bv7/oBwGgRBzLHi%2BACa1AQI8XgDiV/VHH8lyXAqbYoDceLIi9iMnK2BBxiAri0Mu42hX1jrlX8HBLDhHAAKy8H4HBaKQqCcG41jWK2KxrN6sI8KQBCaMTSwANYgFskgvK5AAcGgiwAnAAbK5GgSyLWxbPonCSJT3O05wvAKCAVpc9TxOkHAsBIHcOX0GQFBjqSZtDMAEtJFgABuAmYFBeCYAA7t%2BlJUxzNAlnE2vGer0Tic8nAc6HzDPN%2B0TaOGesc3cWHfgwtBCfrpBYNdRjiJn%2BCEQ4eCO926vkuGxIbBzmk1Or/HRMQTRPEGEe8I81Kt0s5l9gobue97jCtzIggiGI7BSMP8hKGo6u6IEBhGCgjOWPoeDRNrkBLKgyR1NrHAALTYAc%2B/fmYx%2BqtcO%2BXKbdBxPv6G0CQcSXPXCiXCZozMlwzLSt/NN2AnTILgGDuE8G0PQIQwgDHKEMfIaQMgCAmH4OBhRMizEGPEQIACi4CB6OMMBuQsE1EAbgsYfQoFzFgbYMhSC9DTGaOgmBmClgKBZusCQJNOAU1IFTGmdMOAvRFlLfeUtJAHGAMgA6EsXhnxrPgIgxAjjswWG3bmCw%2BYCy2C8CWZMybSilhLSQUtdpkzMBoJIpNVY8PVvwrWOtOZqMNjARAIAbjJGJOQSgN9zYRFYBsVQQiRFiIkVImRvBMDyJIHgOMgR%2BAj1EOICecSp4qHUJnOepAPaN2SJ3ZW5M1aZ34d%2BYk7jPyoC8gE4RojxGSIONI2RHhty30UdsLgKiHH63UaQfmWwzDCy2FLKQcstj6MkNLQWeSrG8N4LY2w9i9ZaC6aTMwBS%2BGaw6YspYJdiDpGcJIIAA%3D)
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
