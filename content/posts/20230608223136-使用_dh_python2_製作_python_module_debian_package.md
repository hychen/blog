+++
title = "使用 dh_python2 製作 Python Module Debian Package"
publishDate = 2011-12-17
categories = ["觀想"]
draft = false
+++

Debian 官方目前建議改用使用dh_python2取代py_support及py_central, 以下是自己轉換的紀錄。 本筆記只適用打包單純的Python module, 並不適合Python module裡面有包含extension.


## debian/control {#debian-control}


### 修改Build-Depends及Build-Depends-Indep，改用python-all取代python-support,python-central {#修改build-depends及build-depends-indep-改用python-all取代python-support-python-central}

```nil
Build-Depends: python-all (>= 2.6.6-3), debhelper (>= 7)
```


### 移除所有的XB-Python-Version，改使用X-Python-Version 指定支援的Python版本 {#移除所有的xb-python-version-改使用x-python-version-指定支援的python版本}

```nil
X-Python-Version: >= 2.5
```


## debian/rules {#debian-rules}

若 debhelper &gt;= 7, 加上 –with python2

```nil
dh $@ --buildsystem=python_distutils --with python2
```

若 debhelper &lt;= 7

-   python-support, 執行 sed -i -e 's/dh_pysupport/dh_python2/' debian/rules
-   python-central, 執行 sed -i -e 's/dh_pycentral/dh_python2/' debian/rules


## 移除不再使用的檔案 {#移除不再使用的檔案}

-   debian/pyversions
-   debian/pycompat


## 參考資料 {#參考資料}

-   dh_python2 manpange
-   [Debian Wiki:TransitionToDHPython2](https://web.archive.org/web/20120910101845/http://wiki.debian.org/Python/TransitionToDHPython2)
-   [Debian Python Policy](https://web.archive.org/web/20120910101845/http://www.debian.org/doc/packaging-manuals/python-policy/)
