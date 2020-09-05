# Issue of descriptor

- global shared state(class attribute)


### Why

![SAMPLE.jpg](.attachments/SAMPLE-a6d2a31d-bbb2-4c8e-9ad5-f563ea297077.jpg)

### Disaster

```python

class PasswordDescriptor:
    access_phrase = '1313'

    
    def __set__(self, instance, value):
        if bool(re.search('@', value)) == False:
            raise ValueError('密碼要包含特殊字元')   
        else:
            self._password = value
            print('密碼設定成功為: ', self._password)

    def __get__(self, instance, owner):
        print("please input access phrase")
        if input() == self.access_phrase:
            return self._password
        else:
            print("403 access deny")
            return None


class Member:
    password = PasswordDescriptor()
    def __init__(self, name, age):
        self._name = name
        self._age = age

```

```shell

>>> member1 = Member('rose', 18)
>>> member1.password = '12873@*hSx'

>>> member2 = Member('alex', 19)
>>> member2.password = 'JSG&2@*HOI'

```

**each member will get same password**: _JSG&2@*HOI_


### How

leverage \_\_dict\_\_
 

### Do the right thing

```python
class PasswordDescriptor:
    access_phrase = '1313'

    
    def __set__(self, instance, value):
        if bool(re.search('@', value)) == False:
            raise ValueError('密碼要包含特殊字元')   
        else:
            instance.__dict__['_password'] = value
            print('密碼設定成功為: ', instance.__dict__['_password'])
      

    def __get__(self, instance, owner):
        print("please input access phrase")
        if input() == self.access_phrase:
            return instance.__dict__['_password']
        else:
            print("403 access deny")
            return None

```










