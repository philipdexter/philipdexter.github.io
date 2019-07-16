---
layout: post
title: First Free Workspace in i3
---

[i3](https://i3wm.org/) is an excellent tiling window manager.
Soon after I started using it I wanted a way to switch to the first
unused workspace. I didn't see any built in way to do this, but I knew
that i3 had python bindings.
The following script switches to the first unused workspace
(only considering the default ones numbered 1 to 10):

```python
#!/usr/bin/env python
"""
Find the first free workspace and switch to it

Add this to your i3 config file:
    bindsym <key-combo> exec python /path/to/this/script.py
"""
import i3

def main():
  workspaces = i3.get_workspaces()
  workints = list()
  for w in workspaces:
    workints.append(w['name'])
  for i in range(1,11):
    if str(i) not in workints:
      i3.workspace(str(i))
      break

if __name__ == '__main__':
  main()
```

As the documentation for the script suggests, I have the following
in my i3 config file to bind the script to win+~:
```
bindcode $mod+49 exec python ~/.i3/firstfree.py
```
