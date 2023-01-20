# Namespace package について

A は non-namespace package であり
namespace-A は nemespace package です。

nomespace-A は A の拡張モジュールを意図されているものとします。

それぞれのケースは、真新しい python の環境で行われているとします。

## Case 1

```bash
pip install -e ./A
cd workspace
python3 -c "import A; print(A.__path__)"
```

というコマンドは

```
['/(省略)/namespace_package/A/A']
```

というようになり、これは namespace_package ではない状況です。

<!-- `A/pyproject.toml` の `namespaces = false` を `namespaces = true` に変えると。 -->


