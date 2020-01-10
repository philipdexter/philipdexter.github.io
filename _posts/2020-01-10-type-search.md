---
layout: post
title: Find Functions by Type in Python
---

After
[adding types to my project]({% post_url 2019-10-31-mypy %}),
I became curious if there was a way to programmatically
query my project's functions based on type.
The
[inspect](https://docs.python.org/3/library/inspect.html)
module has a
[signature](https://docs.python.org/3/library/inspect.html#inspect.signature)
function that returns the annotated type signature,
which makes this possible.

In this example I will query from the `User` module of my `learn` project.
First, I create a list of function names and their signature.

```python
res = []
for f in dir(User):
  x = getattr(User, f)
  if not callable(x):
    continue
  try:
    res.append((f, inspect.signature(x)))
  except:
    continue
```

The value of `res` looks like this
```
[...
('get_setting_value', <Signature (self, name: str) -> Union[str, NoneType]>),
('is_teacher', <Signature (self) -> bool>),
('is_teacher_of_survey', <Signature (self, survey: 'Survey') -> bool>),
('open_surveys', <Signature (self, now: datetime.datetime) -> List[ForwardRef('Survey')]>),
('set_password', <Signature (self, password: str) -> None>),
('set_setting', <Signature (self, name: str, value: str) -> None>),
('started_survey', <Signature (self, *, survey: 'Survey') -> bool>),
('stylize_for_import', <Signature (user: List[str]) -> List[str]>),
('wants_easy_input', <Signature (self) -> bool>),
...]
```

I can now make a function that, for example,
finds all functions that could return a specific type.
By "could return" I mean that the type could be wrapped in a list,
union, or forwardref. There are other possibilities, like sets
and dicts, but I didn't get that far.

```python
def could_return(t):
 ret = []
 for name, sig in res:
   if type_in(t, sig.return_annotation):
     ret.append((name, sig))
 return ret
```

This requires a `type_in` helper function, which we can define
by going over a few cases that I'm interested in.

```python
def type_in(t, tt):
  if t == tt:
    return True
  if hasattr(tt, '__origin__'):
    if tt.__origin__ == list:
      assert len(tt.__args__) == 1
      return type_in(t, tt.__args__[0])
    if tt.__origin__ == typing.Union:
      return any([type_in(t, ttt) for ttt in tt.__args__])
  if tt.__class__ == typing.ForwardRef:
    return type_in(t, tt.__forward_arg__)
  return False
```

The code is a little awkward since there is no clean type inspection API that I
know of.

Now, we can query
```python
>>> could_return(bool)
[...
('is_teacher', <Signature (self) -> bool>),
('is_teacher_of_survey', <Signature (self, survey: 'Survey') -> bool>),
('started_survey', <Signature (self, *, survey: 'Survey') -> bool>),
('wants_easy_input', <Signature (self) -> bool>),
...]
>>> could_return('Survey')
[...
('accessible_surveys', <Signature (self) -> List[ForwardRef('Survey')]>),
...]
>>> could_return(str)
[...
('get_setting_value', <Signature (self, name: str) -> Union[str, NoneType]>),
('stylize_for_import', <Signature (user: List[str]) -> List[str]>),
...]
```
