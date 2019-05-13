**GIT revert multiple commits**

Дано:
```
A <-- B  <-- C <-- D <-- master <-- HEAD
```

Хотим:
```
A <-- B  <-- C <-- D <-- [(CD)^-1] <-- master <-- HEAD
```

Решение:
```
$ git revert --no-commit D
$ git revert --no-commit C
$ git commit -m "the commit message"

еще это можно решить так:
$ git checkout -f B -- .
$ git commit -a
```

---

**GIT отделить ветку в отдельный репозиторий**

Вариант 1: 

```
git push url://to/new/repository.git branch-to-move:new-branch-name
```

Вариант 2:

```
git clone -b newbranch CurrentRepo NewRepo
```

---

