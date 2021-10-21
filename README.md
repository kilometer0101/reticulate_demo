---
title: "Reticulate demo"
output:
  html_document:
    keep_md: TRUE
---



# 資料

前にTokyo.Rで話した[スライド](https://speakerdeck.com/kilometer/tokyo-dot-r-number-80-r-interface-to-python)。

# 準備

1. Rをインストール
2. Pythonをインストール
3. RStudioをインストール
4. RStudioでプロジェクトを作る
5. RStudioのプロジェクトのTerminalを開く
6. pipenvをインストール `brew install pipenv`
7. 仮想環境を作る `pipenv shell`
8. numpyをインストール `pipenv install numpy`
9. 仮想環境のアドレスを確認 `pipenv --venue`

pandasは人権なのでインストールしておこう `pipenv install pandas`

# 環境設定

## R側


```r
library(tidyverse)
library(reticulate)
library(palmerpenguins)
```



## Python環境を指定

`pipenv --venv`の出力結果に`/bin/python`をくっつけたもの。


```r
pyenv <- "/Users/[user_name]/.local/share/virtualenvs/[project_name]-[randum_strings]/bin/python"

use_python(python = pyenv,
           required = TRUE)
```

## Python側


```python
import numpy as np
import pandas as pd
```

# 挙動

Rコードは{r}のチャンクで書く。
こんな感じ、

    ```{r}
    df <- penguins
    ```




RのオブジェクトをPython側で読み込む時は`r.`をつけるだけ

    ```{python}
    df = r.df
    ```



普通にPythonが書ける。


```python
df
```

```
##        species     island  bill_length_mm  ...  body_mass_g     sex  year
## 0       Adelie  Torgersen            39.1  ...         3750    male  2007
## 1       Adelie  Torgersen            39.5  ...         3800  female  2007
## 2       Adelie  Torgersen            40.3  ...         3250  female  2007
## 3       Adelie  Torgersen             NaN  ...  -2147483648     NaN  2007
## 4       Adelie  Torgersen            36.7  ...         3450  female  2007
## ..         ...        ...             ...  ...          ...     ...   ...
## 339  Chinstrap      Dream            55.8  ...         4000    male  2009
## 340  Chinstrap      Dream            43.5  ...         3400  female  2009
## 341  Chinstrap      Dream            49.6  ...         3775    male  2009
## 342  Chinstrap      Dream            50.8  ...         4100    male  2009
## 343  Chinstrap      Dream            50.2  ...         3775  female  2009
## 
## [344 rows x 8 columns]
```

dplyrの`filter()`はpandasの`query()`


```python
df_adelie = df.query('species == "Adelie"')
```

dplyrの`select`はpandasの`filter()`


```python
df_bill_body = df_adelie.filter(["bill_length_mm", "body_mass_g"])
```

R側にPythonのオブジェクトを読み込む時


```r
py <- import_main()
df_a <- py$df_bill_body

df_a %>% 
  filter(bill_length_mm >= 40) %>% 
  as_tibble()
```

```
## # A tibble: 51 x 2
##    bill_length_mm body_mass_g
##             <dbl>       <int>
##  1           40.3        3250
##  2           42          4250
##  3           41.1        3200
##  4           42.5        4500
##  5           46          4200
##  6           40.6        3550
##  7           40.5        3200
##  8           40.5        3950
##  9           40.9        3900
## 10           42.2        3550
## # … with 41 more rows
```

## Numpyの受け渡し

numpyのアレイはR側にはマトリックスで渡される。


```python
x = [0, 1, 2]
y = np.array([range(5)])
z = np.array([range(15)]).reshape(5, 3)
```



```r
py <- import_main()

py$x
```

```
## [1] 0 1 2
```

```r
py$y
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    0    1    2    3    4
```

```r
py$z
```

```
##      [,1] [,2] [,3]
## [1,]    0    1    2
## [2,]    3    4    5
## [3,]    6    7    8
## [4,]    9   10   11
## [5,]   12   13   14
```

逆もまた然り。


```r
xx <- c(0:4)
yy <- matrix(0:14, 3)
```


```python
r.xx
```

```
## [0, 1, 2, 3, 4]
```

```python
r.yy
```

```
## array([[ 0,  3,  6,  9, 12],
##        [ 1,  4,  7, 10, 13],
##        [ 2,  5,  8, 11, 14]], dtype=int32)
```

