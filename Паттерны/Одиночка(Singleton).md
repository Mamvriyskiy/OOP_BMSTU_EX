>Подойдет, если нам необходимо иметь во всей системе объект только в единственном экземпляре.

Преимущества:
- гарантирует наличие единственного экземпляра класса
- предоставляет доступ из любой части программы

Недостатки:
- анти-паттерн, так как представляет собой глобальный объект.
- не можем принимать решение о том, какой объект создавать при работе программы. Выполняется на этапе компиляции.

Альтернатива: фабричный метод, который создает объект только один раз. 

```c++
# include <iostream>
# include <memory>

using namespace std;

class Product
{
public:
	static shared_ptr<Product> instance()
	{
		class Proxy : public Product {};
		
		static shared_ptr<Product> myInstance = make_shared<Proxy>();
		
		return myInstance;
	}
	
	~Product() { cout << "Calling the destructor;" << endl; }
	
	void f() { cout << "Method f;" << endl; }
	
	Product(const Product&) = delete;
	Product& operator =(const Product&) = delete;

private:
	Product() { cout << "Calling the default constructor;" << endl; }
};

int main()
{
	shared_ptr<Product> ptr(Product::instance());

	ptr->f();
}

```