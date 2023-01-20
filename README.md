# Namespace package について

A は non-namespace package であり
namespace-A は nemespace package です。

nomespace-A は A の拡張モジュールを意図されているものとします。

それぞれのケースは、真新しい python の環境で行われているとします。

ディレクトリ構成は

- A
```
A
├── A
│   ├── __init__.py
│   └── foo.py
└── pyproject.toml
```

- namespace-A
```
namespace-A
├── A
│   └── B
│       └── C.py
└── pyproject.toml
```

## 動作確認
```bash
pip install --quiet ./A ./namespace-A
cd workspace
echo "******** Parent Package *********"
python3 -c "import A;A.foo.test_print();"
echo "******** Namespace Package *********"
python3 -c "from A.B.C import foo;foo();"
```

## Case 1

```bash
pip freeze > tmp.txt; pip uninstall -y -r tmp.txt; rm tmp.txt
pip install --quiet ./namespace-A
pip install --quiet -e ./A
cd workspace
echo "******** Parent Package *********"
python3 -c "import A;A.foo.test_print();"
echo "******** Namespace Package *********"
python3 -c "from A.B.C import foo;foo();"
```

この場合は、

python3 -c "import A;print(A.__path__)"
