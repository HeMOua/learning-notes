# VSCode相对路径

```python
import os, sys

os.environ['KMP_DUPLICATE_LIB_OK'] = 'TRUE'
os.chdir(sys.path[0])
```

