>Основная идея заключается в том, чтобы избавиться от явного создания объектов. Эти будут заниматься «креаторы». При этом то, какой конкретно объект будет создан, т.е. то, какой креатор использовать будут говорить нам особенная таблица. Иными словами, будет выделяться отдельная сущность, которая будет принимать решение о том, какой объект создавать.

>Это дает возможность выбирать объект, который создавать, во время выполнения программы. Так же, во время выполнения программы, мы можем подменять создание одного объекта, на создание другого.

Преимущества:
- создание в коде конкретных объектов
- четко отделяем принятие решения о создании объекта от создания самого объекта. Выделяется целая сущность, которая принимает решение, какой объект создавать.
- во время выполнения программы решать какой объект создавать.
- во время выполнения программы мо можем подменять один объект на другой.
- повторное использование одних и тех же объектов. 

Недостатки:
- возрастает количество кода
- резко растет время выполнения
- можно сделать на шаблонах, но после одного изменения будет все перекомпилировано 

![[fp.png]]

#### Пример 1. Создание нового объекта.
```c++
# include <iostream>
# include <memory>

using namespace std;

# pragma region Product
class Product
{
public:
	virtual ~Product() = default;

	virtual void run() = 0;
};

class ConProd1 : public Product
{
public:
	ConProd1() { cout << "Calling the ConProd1 constructor;" << endl; }
	~ConProd1() override { cout << "Calling the ConProd1 destructor;" << endl; }

	void run() override { cout << "Calling the run method;" << endl; }
};

# pragma endregion

class Creator
{
public:
	virtual ~Creator() = default;

	virtual unique_ptr<Product> createProduct() = 0;
};

template <typename Derived, typename Base>
concept Derivative = is_abstract_v<Base> && is_base_of_v<Base, Derived>;

template <Derivative<Product> Tprod>
class ConCreator : public Creator
{
public:
	unique_ptr<Product> createProduct() override
	{
	return unique_ptr<Product>(new Tprod());
	}
};

class User
{
public:
	void use(shared_ptr<Creator>& cr)
	{
	shared_ptr<Product> ptr = cr->createProduct();

	ptr->run();
	}
};

int main()
{
	shared_ptr<Creator> cr = make_shared<ConCreator<ConProd1>>();

	unique_ptr<User> us = make_unique<User>();

	us->use(cr);
}
```

#### Пример 2.  Без повторного создания.

Один раз создав продукт, в дальнейшем мы его можем только возвращать.

 ```c++
 # include <iostream>
# include <memory>

using namespace std;

class Product;

class Creator
{
public:
	virtual ~Creator() = default;
	shared_ptr<Product> getProduct();

protected:
	virtual shared_ptr<Product> createProduct() = 0;

private:
	shared_ptr<Product> product;
};

template <derived_from<Product> Tprod>
class ConCreator : public Creator
{
protected:
	shared_ptr<Product> createProduct() override
	{
	return  make_shared<Tprod>();
	}
};

# pragma region Method Creator
shared_ptr<Product> Creator::getProduct()
{
	if (!product)
	{
		product = createProduct();
	}

	return product;
}

# pragma endregion

# pragma region Product
class Product
{
public:
	virtual ~Product() = default;

	virtual void run() = 0;
};

class ConProd1 : public Product
{
public:
	ConProd1() { cout << "Calling the ConProd1 constructor;" << endl; }
	~ConProd1() override { cout << "Calling the ConProd1 destructor;" << endl; }

	void run() override { cout << "Calling the run method;" << endl; }
};

# pragma endregion

int main()
{
	shared_ptr<Creator> cr = make_shared<ConCreator<ConProd1>>();
	shared_ptr<Product> ptr1 = cr->getProduct();
	shared_ptr<Product> ptr2 = cr->getProduct();

	cout << "use count = " << ptr1.use_count() << endl;
	ptr1->run();
}

```

#### Пример 3. Разделение обязанностей.
```c++
# include <iostream>
# include <initializer_list>
# include <memory>
# include <map>

using namespace std;

class Product;

class Creator
{
public:
	virtual ~Creator() = default;

	virtual unique_ptr<Product> createProduct() = 0;
};

template <derived_from<Product> Tprod>
class ConCreator : public Creator
{
public:
	unique_ptr<Product> createProduct() override
	{
		return make_unique<Tprod>();
	}
};

# pragma region Product
class Product
{
public:
	virtual ~Product() = default;

	virtual void run() = 0;
};

class ConProd1 : public Product
{
public:
	ConProd1() { cout << "Calling the ConProd1 constructor;" << endl; }
	~ConProd1() override { cout << "Calling the ConProd1 destructor;" << endl; }

	void run() override { cout << "Calling the run method ConProd1;" << endl; }
};

class ConProd2 : public Product
{
public:
	ConProd2() { cout << "Calling the ConProd2 constructor;" << endl; }
	~ConProd2() override { cout << "Calling the ConProd2 destructor;" << endl; }

	void run() override { cout << "Calling the run method ConProd2;" << endl; }
};
# pragma endregion

class CrCreator
{
public:
	template <typename Tprod>
	static unique_ptr<Creator> createConCreator()
	{
		return make_unique<ConCreator<Tprod>>();
	}
};

class Solution
{
	using CreateCreator = unique_ptr<Creator>(&)();
	using CallBackMap = map<size_t, CreateCreator>;

public:
	Solution() = default;
	Solution(initializer_list<pair<size_t, CreateCreator>> list);

	bool registration(size_t id, CreateCreator createfun);
	bool check(size_t id) { return callbacks.erase(id) == 1; }

	unique_ptr<Creator> create(size_t id);

private:
	CallBackMap callbacks;
};

# pragma region Solution
Solution::Solution(initializer_list<pair<size_t, CreateCreator>> list)
{
	for (auto&& elem : list)
	this->registration(elem.first, elem.second);
}

bool Solution::registration(size_t id, CreateCreator createfun)
{
	return callbacks.insert(CallBackMap::value_type(id, createfun)).second;
}

unique_ptr<Creator> Solution::create(size_t id)
{
	CallBackMap::const_iterator it = callbacks.find(id);

	if (it == callbacks.end())
	{
	//throw IdError();
	}

	return unique_ptr<Creator>(it->second());
}

# pragma endregion

int main()
{
	shared_ptr<Solution> solution(new Solution({ {1, CrCreator::createConCreator<ConProd1>} }));

	if (!solution->registration(2, CrCreator::createConCreator<ConProd2>))
	{
		cout << "Error registration!" << endl;
	// throw ...
	}
	else
	{
		solution->registration(2, CrCreator::createConCreator<ConProd2>);
		
		shared_ptr<Creator> cr(solution->create(2));
		shared_ptr<Product> ptr = cr->createProduct();
		
		ptr->run();
	}
}
```

