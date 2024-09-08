---
tags:
  - shellscript
  - code
create: 2024-08-04 20:07:19
---


```sh
#!/bin/bash

SCRIPT_DIR=$(dirname "$0")
python "$SCRIPT_DIR/create.py"
```

이렇게 하면 스크립트가 실행되는 위치에 관계없이 `create.py` 파일을 올바르게 참조하게 돼. 다시 시도해보고 제대로 작동하는지 확인해줘.