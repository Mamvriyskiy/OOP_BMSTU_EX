Дает возможность сразу подменить одну группу создания объектов на другую. 

Преимущества:
- создание в коде конкретных объектов
- четко отделяем принятие решения о создании объекта от создания самого объекта. Выделяется целая сущность, которая принимает решение, какой объект создавать.
- во время выполнения программы решать какой объект создавать.
- во время выполнения программы мо можем подменять один объект на другой.
- повторное использование одних и тех же объектов. 
- не нужно контролировать создание объектов из разных семейств

Недостатки:
- у абстрактной фабрики должен быть базовый интерфейс, если у библиотек используются различные понятия, то выделить базовое для всех не всегда возможно.
- не дает возможности одновременно создавать объекты из разныъ семейств(только из одного семейства).
- когда объединям в один класс разные creator возникает необходимость, чтобы в разных иерархиях были подобные сущности для разных ситуаций.

![[af.png]]

```c++
# include <iostream>
# include <memory>

using namespace std;

class Image {};
class Color {};

class BaseGraphics
{
public:
	virtual ~BaseGraphics() = 0;
};

BaseGraphics::~BaseGraphics() {}

class BasePen {};
class BaseBrush {};

class QtGraphics : public BaseGraphics
{
public:
	QtGraphics(shared_ptr<Image> im) { cout << "Calling the QtGraphics constructor;" << endl; }
	~QtGraphics() override { cout << "Calling the QtGraphics destructor;" << endl; }
};

class QtPen : public BasePen {};
class QtBrush : public BaseBrush {};

class AbstractGraphFactory
{
public:
	virtual ~AbstractGraphFactory() = default;

	virtual unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) = 0;
	virtual unique_ptr<BasePen> createPen(shared_ptr<Color> cl) = 0;
	virtual unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) = 0;
};

class QtGraphFactory : public AbstractGraphFactory
{
public:
	unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) override
	{
		return make_unique<QtGraphics>(im);
	}

	unique_ptr<BasePen> createPen(shared_ptr<Color> cl) override
	{
		return make_unique<QtPen>();
	}

	unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) override
	{
		return make_unique<QtBrush>();
	}
};

int main()
{
	shared_ptr<AbstractGraphFactory> grfactory = make_shared<QtGraphFactory>();

	shared_ptr<Image> image = make_shared<Image>();

	shared_ptr<BaseGraphics> graphics1 = grfactory->createGraphics(image);
}
```