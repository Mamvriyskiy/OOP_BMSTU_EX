>Позволяет создавать объекты на основе других, не вызывая creator.

>Мы добавляем в базовый класс метод clone(), возвращающий указатель на себя, производные классы реализуют clone() под себя, возвращая указатель на подобный объект.

Преимущества:
- Ускоряет создание объектов
- Позволяет клонировать объекты, не привязывая их к конкретным классам

Недостатки:
- тяжело клонировать составные объекты, имеющие ссылки на другие объекты, а так же объекты, которые во внутреннем представлении имеют другие объекты

![[pr.png]]

```c++
# include <iostream>
# include <memory>

using namespace std;

class BaseObject
{
public:
	virtual ~BaseObject() = default;

	virtual unique_ptr<BaseObject> clone() = 0;
};

class Object1 : public BaseObject
{
public:
	Object1() { cout << "Calling the default constructor;" << endl; }
	Object1(const Object1& obj) { cout << "Calling the Copy constructor;" << endl; }
	~Object1() override { cout << "Calling the destructor;" << endl; }

	unique_ptr<BaseObject> clone() override
	{
	return make_unique<Object1>(*this);
	}
};

int main()
{
	shared_ptr<BaseObject> ptr1 = make_shared<Object1>();

	auto ptr2 = ptr1->clone();
}

```