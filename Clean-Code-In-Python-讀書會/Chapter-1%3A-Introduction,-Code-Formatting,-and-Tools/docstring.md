# docstring 風格
- Epytext
```
This is a javadoc style.

@param param1: this is a first param
@param param2: this is a second param
@return: this is a description of what is returned
@raise keyError: raises an exception
```
- reST
```
This is a reST style.

:param param1: this is a first param
:param param2: this is a second param
:returns: this is a description of what is returned
:raises keyError: raises an exception
```

- Google
```
This is a groups style docs.

Parameters:
    param1 - this is the first param
    param2 - this is a second param

Returns:
    This is a description of what is returned

Raises:
    KeyError - raises an exception
```

- Numpydoc
```
My numpydoc description of a kind
of very exhautive numpydoc format docstring.

Parameters
----------
first : array_like
    the 1st param name `first`
second :
    the 2nd param
third : {'value', 'other'}, optional
    the 3rd param, by default 'value'

Returns
-------
string
    a value in a string

Raises
------
KeyError
    when a key error
OtherError
    when an other error
```

# 查詢 docstring
- 使用 help
![image.png](.attachments/image-e8087e49-9d48-4499-81d4-68fa3806a932.png)

- 使用 __doc__
![image.png](.attachments/image-2cfd66c2-e65f-4c25-8f57-266d029a124d.png)

- 在 jupyter 中按 shift + Tab
![image.png](.attachments/image-4573cca0-7900-4314-9bfe-ac02beeeeb81.png)