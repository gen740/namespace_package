# Namespace package について

A は non-namespace package であり
namespace-A は nemespace package です。

namespace-A は A の拡張モジュールを意図されているものとします。

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
cd ..
```

## Case 1

```bash
git clean -xffd
pip freeze > tmp.txt; pip uninstall -y -r tmp.txt; rm tmp.txt
pip install --quiet ./namespace-A
pip install --quiet -e ./A
cd workspace
echo "******** Parent Package *********"
python3 -c "import A;A.foo.test_print();"
echo "******** Namespace Package *********"
python3 -c "from A.B.C import foo;foo();"
cd ..
```

parent package の A.foo.test_print(); が使用できない。
pip install の順番によらず、これはおこる。

`python3 -c "import A.foo;A.foo.test_print();"`

は可能である。

`python3 -c "import A.foo;print(A.__path__)"`

の結果は
`_NamespacePath(['(省略)/lib/python3.10/site-packages/A'])`
を返し、これは `namespace-A` を参照している。

## Case 2
```bash
git clean -xffd
pip freeze > tmp.txt; pip uninstall -y -r tmp.txt; rm tmp.txt
pip install --quiet -e ./namespace-A
pip install --quiet -e ./A
cd workspace
echo "******** Parent Package *********"
python3 -c "import A;A.foo.test_print();"
echo "******** Namespace Package *********"
python3 -c "from A.B.C import foo;foo();"
cd ..
```

parent package の A.foo.test_print(); が使用できない。
どちらとも、 editable 出あっても、 parent は使用ができない

こちらも `python3 -c "import A.foo;A.foo.test_print();"`
が可能である。

## Case 3
```bash
git clean -xffd
pip freeze > tmp.txt; pip uninstall -y -r tmp.txt; rm tmp.txt
pip install --quiet -e ./namespace-A
pip install --quiet ./A
cd workspace
echo "******** Parent Package *********"
python3 -c "import A;A.foo.test_print();"
echo "******** Namespace Package *********"
python3 -c "from A.B.C import foo;foo();"
```

この場合は、parent package の `A.B.C.foo.foo()` が使用できない。

## 結論
Parent と その拡張はどちらも、 editable ではなくインストールするべきである。
| editable install | 状況                            |
| -------------    | -------------                   |
| しない           | 両方とも意図した通りに使える    |
| 両方             | `import A;A.foo...;` が使えない |
| parent           | `import A;A.foo...;` が使えない |
| extention        | `import A.B.C ...;` が使えない  |

`import A` の段階では、 `parent` か `extention` のどちらに、
パスを解決するか判断できないために、これが起こると考えられる。
