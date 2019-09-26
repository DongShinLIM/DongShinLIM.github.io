---
title: 'Basic python grammar for text mining'
date: 2019-09-26 00:00:00
featured_image: '/images/demo/text.jpg'
excerpt: 파이썬을 이용한 텍스트 마이닝을 할 때 자주 사용되는 기초 문법 정리
---

### 텍스트 마이닝을 위한 파이썬 기초 문법 정리
#### 제거 및 삽입

```python
# 내부 변수를 지정하여 제거할 수 있다.
x = [1, 2, 3, 4, 5]
y = ["a", "b", "c"]
```


```python
x.remove(3)
x

```




    [1, 2, 4, 5]




```python
y.remove("a")
y
```




    ['b', 'c']




```python
# 현재 index == 1인 것의 앞에 10을 넣겠다는 의미
x.insert(2, 10)
x
```




    [1, 2, 10, 4, 5]




```python
# 현재 index == -1인 것 앞에 20을 넣겠다는 의미
x.insert(-1, 20)
x
```




    [1, 2, 10, 4, 20, 5]




```python
# 현재 존재 하지 않는 index 값을 지정하면 가장 마지막으로 들어간다.
x.insert(6, 30)
x
```




    [1, 2, 10, 4, 20, 5, 30]



### 정렬 (sort와 sorted 의 차이는 원본 영향을 주는지 여부이다)


```python
# sort를 이용하면 원본 리스트를 정렬하되 반환값은 None이다.
# 원본 리스트에 순서를 변경한다. (원본 리스트에 영향 있음)
x = [1,5,2,30,10]
x.sort(reverse = False)
x
```


```python
print(x.sort())
```

    None



```python
# sorted 함수는 정렬된 새로운 리스트를 반환한다. (원본 리스트에는 영향 없음)
# 모든 iterable에 동작한다. (list, tupel, dict, string 등)
y = [1, 10, 4, 20, 3, 30]
sorted(y)

```




    [1, 3, 4, 10, 20, 30]




```python
y
```




    [1, 10, 4, 20, 3, 30]



### "in" operator를 이용해서 value 확인


```python
100 in y
```




    False




```python
1 in y
```




    True



### 대소 비교
#### string과 숫자는 대소 비교 불가능


```python
a = ['z', 6, 4]
```


```python
min(a)
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-65-7deb7b701666> in <module>
    ----> 1 min(a)
    

    TypeError: '<' not supported between instances of 'int' and 'str'



```python
a = [2, 6, 4]
min(a)
```




    2



string 끼리 대소 비교 가능 - ASCII 코드로 비교


```python
b = ['a', 'b', 'c']
min(b)
```




    'a'




```python
# ord - ASCII 코드 출력
ord('a')
```




    97




```python
# chr - ASCII 코드 입력하면 결과 출력
chr(97)
```




    'a'




```python
c = ['ㄱ', 'ㄴ', 'ㄷ']
min(c)
```




    'ㄱ'




```python
# len - 길이 리턴
b[len(b) - 1]
```




    'c'



#### exercise


```python
test = [3, "Data Science", "5"]
test[0]
```




    3




```python
# '숫자'는 string으로 인식한다.
type(test[2])
```




    str




```python
len(test)
```




    3




```python
# 하나의 값을 추가할 때
test.append(6)
```


```python
# 두 개 이상의 값을 추가 할 때
test.extend([7, 8])
print(test)
```

### String


```python
s = 'python'
s[0]
```


```python
# slicing을 하면 문자열의 형태로 출력
s[0:2]
```




    'py'




```python
# slicing 상태에서 바로 추출 가능
s[0:2][-1]
```




    'y'




```python
# slicing으로 문자열에 새로운 값 대입 불가능 (immutable data type)
s[0] = 'c'
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-55-9eb09e0dc58b> in <module>
          1 # slicing으로 문자열에 새로운 값 대입 불가능 (immutable data type)
    ----> 2 s[0] = 'c'
    

    TypeError: 'str' object does not support item assignment



```python
s = 'Today is a good day'
```

공백문자(whitespace chacter)  
- 띄어쓰기
- 탭
- 줄바꿈(new-line, enter)


```python
# 문자열은 immutable 이므로 split() 함수를 사용해도 바뀌지 않음
s.split()
```




    ['Today', 'is', 'a', 'good', 'day']




```python
words = s.split()
words[0]
```


```python
s.split()[-1]
```




    'day'




```python
type(words[-1])
```




    str




```python
# 특정 문자를 기준으로 split()도 가능하다.
s.split('a')
```




    ['Tod', 'y is ', ' good d', 'y']




```python
# strip() 양쪽 끝에 존재하는 것들만 제거, 함수에 인자를 전달하지 않으면 공백문자만 제거된다.
s.strip()
```




    'Today is a good day'




```python
t = '\t Today is a good day \t'
print(t)
```

    	 Today is a good day 	



```python
print(t.strip())
```

    Today is a good day



```python
# 끝에 존재하는 것이 아니면 아무 것도 제거되지 않는다.
s.strip('good')
```




    'Today is a good day'



#### backslash

`\t` - tap에 해당  
`\n` - 줄바꿈에 해당


```python
s1 = 'today \t tomorrow \n yesterday'
```


```python
print(s1)
```

    today	 tomorrow
     yesterday



```python
h = '\tI like data science.\n'
print(h)
```


```python
# \t와 \n 모두 공백문자이기 때문에 strip()으로 제거됨
print(h.strip())
```

    I like data science.


`lstrip()`, `rstrip()` - 한 쪽만 제거


```python
# find()는 매치 되는 첫 str index를 반환한다.
s.find('is')
```




    6




```python
# 없는 단어의 경우 -1 리턴
s.find('was')
```




    -1




```python
# replace(기존 문자, 대체할 문자)는 해당되는 모든 문자를 대체한다.
s.replace("o", "a")
```




    'Taday is a gaad day'




```python
s1 = "Trump is the president of U.S. and US is a country in America"
```


```python
# 단어 간 대체 가능
s2 = s1.replace("U.S.", "US")
s2
```


```python
h
```




    '\tI like data science.\n'




```python
# 공백문자도 대체 가능하다.
h.replace("\t", "")
```




    'I like data science.\n'




```python
# 함수를 연이어 사용 할 수 있다.
h.replace("\t", "").replace("\n", "")
```




    'I like data science.'




```python
h.count("i")
```




    2




```python
# 리스트 데이터를 join()을 사용하여 원하는 문자로 연결 
" vs ".join(["Korea", "Japan", "China"])
```




    'Korea vs Japan vs China'




```python
# CSV
" , ".join(["Korea", "Japan", "China"])
```




    'Korea , Japan , China'




```python
### csv 파일 저장하기
```


```python
data = ' vs '.join(["Korea", "Japan", "China"])
f = open('test.csv', 'w')
f.write(data)
f.close()
r = open('test.csv', 'r')
r.read()
```




    'Korea vs Japan vs China'



#### use of backslash

\\', \\" - 문자열 내에서 작은따옴표, 큰따옴표 사용 시


```python
k = 'Tom's book'
```


      File "<ipython-input-80-5b72114440fd>", line 1
        k = 'Tom's book'
                 ^
    SyntaxError: invalid syntax




```python
k = 'Tom\'s book'
k
```


```python
k1 = "Jonathan's book"
k1
```


```python
#### 숫자와 문자의 덧셈, 곱셈에 대한 이해
```


```python
b = '100'
b * 3
```




    '100100100'




```python
str(int(b) * 3)
```




    '300'




```python
float(b) * 3
```




    300.0




```python
b * 10
```




    '100100100100100100100100100100'



#### exercise


```python
new = '\tToday is Monday. I am waiting for the weekend.\n'
print(new)
```

    	Today is Monday. I am waiting for the weekend.
    



```python
print(new.strip())
```

    Today is Monday. I am waiting for the weekend.



```python
print(new.replace('T', 'C'))
```

    	Coday is Monday. I am waiting for the weekend.
    



```python
new.split()
```




    ['Today', 'is', 'Monday.', 'I', 'am', 'waiting', 'for', 'the', 'weekend.']




```python
len(new)
```




    48



### Dictionary

{Key: Value}


```python
dict1 = {'Tom': 23, 'John': 34, 'Bob': 12}
dict1['John']
```




    34




```python
# 새로운 key:value 추가 가능
dict1['Sarah'] =33
dict1
```




    {'Tom': 23, 'John': 34, 'Bob': 12, 'Sarah': 33}




```python
# 동일한 이름을 갖는 Key가 복수인 경우, 가장 나중에 입력된 것으로만 저장됨
dict2 = {'Tom': 23, 'John': 34, 'Bob': 12, 'Tom': 43}
dict2
```




    {'Tom': 43, 'John': 34, 'Bob': 12}




```python
dict2.keys()
```




    dict_keys(['Tom', 'John', 'Bob'])




```python
dict2.values()
```




    dict_values([43, 34, 12])




```python
list(dict2.keys())[-1]
```




    'Bob'




```python
'Bob' in dict2.keys()
```




    True




```python
# update()를 통해 dict 추가 가능
d1 = {'Tom': 23, 'John': 34, 'Bob': 12}
d2 = {'Kai': 28}
d1.update(d2)
d1
```




    {'Tom': 23, 'John': 34, 'Bob': 12, 'Kai': 28}




```python
# update 하려고 할 때 중복되는 Key가 있는 경우, 새롭게 업데이트 되는 dict 해당 값으로 대체
d1 = {'Tom': 23, 'John': 34, 'Bob': 12}
d3 = {'Kai': 28, 'John': 45}
d1.update(d3)
d1
```




    {'Tom': 23, 'John': 45, 'Bob': 12, 'Kai': 28}



#### exercise


```python
# value 값을 기준으로 정렬하기
d = {'Ethan': 7, 'John': 31, 'Kai': 12, 'Peyton': 10, 'Sarah': 28, 'Tom': 40}
d.items()
```




    dict_items([('Ethan', 7), ('John', 31), ('Kai', 12), ('Peyton', 10), ('Sarah', 28), ('Tom', 40)])




```python
# x에는 key, value가 들어있는데 value는 두 번째 원소이므로 x[1]로 인덱싱
sorted(d.items(), key = lambda x: x[1])
```




    [('Ethan', 7),
     ('Peyton', 10),
     ('Kai', 12),
     ('Sarah', 28),
     ('John', 31),
     ('Tom', 40)]




```python
sorted(d.items(), key = lambda x:x[1], reverse = True)
```




    [('Tom', 40),
     ('John', 31),
     ('Sarah', 28),
     ('Kai', 12),
     ('Peyton', 10),
     ('Ethan', 7)]




```python
xyz = [10, 3, 5, 2, 66, 23]
sorted(xyz, key = lambda x: x, reverse = True)
```




    [66, 23, 10, 5, 3, 2]



#### exercise


```python
age_pair = {'Tom': 20}
age_pair['Tom']
```




    20




```python
age_pair['John'] = 30
age_pair
```




    {'Tom': 20, 'John': 30}




```python
len(age_pair.keys())
```




    2




```python
two = {'Sarah': 28, 'Jack': 41}
age_pair.update(two)
age_pair
```




    {'Tom': 20, 'John': 30, 'Sarah': 28, 'Jack': 41}




```python
age_pair.keys()
```




    dict_keys(['Tom', 'John', 'Sarah', 'Jack'])



### Tuple
바뀌지 않는 데이터셋 생성에 이용

### Set
중복제거 목적으로 이용


```python
a = [1, 1, 2, 2, 2, 3]
set(a)
```




    {1, 2, 3}




```python
len(set(a))
```




    3




```python
t = 'today today yesterday yesterday march may may'
# 공백으로 나뉘어져 있는 전체 단어의 개수
len(t.split())
```




    7




```python
# unique한 단어의 개수는 set으로 중복을 제거 한 뒤 len()
len(set(t.split()))
```




    4



### Control Flow

#### if-else


```python
a = 3
if a > 0:
    print('this is the body of if')
```

    this is the body of if



```python
# 조건 불만족으로 print() 되는 값이 없다.
a = -2
if a > 0:
    print('this is the body of if')
    print(a)

```


```python
a = -2
if a > 0:
    print('this is the body of if')
    print(a)
else:
    print('the number is not positive')
```

    the number is not positive



```python
a = -2
if a > 0:
    print('the number is positive')
# elif - if와 else 사이에 위치
elif a == 0:
    print('the number is zero')
else:
    print('the number is negative')
```

    the number is negative



```python
a = 0
if a > 0:
    print('the number is positive')
# elif - if와 else 사이에 위치
elif a == 0:
    print('the number is zero')
else:
    print('the number is negative')
```

    the number is zero

