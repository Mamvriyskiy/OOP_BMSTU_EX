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
- не дает возможности одновременно создавать объекты из разных семейств(только из одного семейства).
- когда объединяем в один класс разные creator возникает необходимость, чтобы в разных иерархиях были подобные сущности для разных ситуаций.

![[af.png]]

```c++
# include <iostream>
# include <memory>

using namespace std;

// Объявления классов Image и Color
class Image {};
class Color {};

// Абстрактный базовый класс для графических элементов
class BaseGraphics
{
public:
	virtual ~BaseGraphics() = 0; // Чисто виртуальный деструктор для абстрактности
};

BaseGraphics::~BaseGraphics() {} // Реализация пуста, так как это чисто виртуальный деструктор

// Объявления базовых классов для пера и кисти
class BasePen {};
class BaseBrush {};

// Класс для графики Qt, производный от BaseGraphics
class QtGraphics : public BaseGraphics
{
public:
	QtGraphics(shared_ptr<Image> im) { cout << "Calling the QtGraphics constructor;" << endl; }
	~QtGraphics() override { cout << "Calling the QtGraphics destructor;" << endl; }
};

// Классы QtPen и QtBrush, производные от BasePen и BaseBrush соответственно
class QtPen : public BasePen {};
class QtBrush : public BaseBrush {};

// Абстрактная фабрика для создания графических элементов, пера и кисти
class AbstractGraphFactory
{
public:
	virtual ~AbstractGraphFactory() = default;

	// Фабричный метод для создания графического элемента
	virtual unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) = 0;

	// Фабричный метод для создания пера
	virtual unique_ptr<BasePen> createPen(shared_ptr<Color> cl) = 0;

	// Фабричный метод для создания кисти
	virtual unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) = 0;
};

// Конкретная реализация фабрики для библиотеки Qt
class QtGraphFactory : public AbstractGraphFactory
{
public:
	// Создание объекта QtGraphics с передачей изображения
	unique_ptr<BaseGraphics> createGraphics(shared_ptr<Image> im) override
	{
		return make_unique<QtGraphics>(im);
	}

	// Создание объекта QtPen (пока без параметров)
	unique_ptr<BasePen> createPen(shared_ptr<Color> cl) override
	{
		return make_unique<QtPen>();
	}

	// Создание объекта QtBrush (пока без параметров)
	unique_ptr<BaseBrush> createBrush(shared_ptr<Color> cl) override
	{
		return make_unique<QtBrush>();
	}
};

int main()
{
	// Создание фабрики QtGraphFactory
	shared_ptr<AbstractGraphFactory> grfactory = make_shared<QtGraphFactory>();

	// Создание объекта изображения
	shared_ptr<Image> image = make_shared<Image>();

	// Использование фабрики для создания графического элемента
	shared_ptr<BaseGraphics> graphics1 = grfactory->createGraphics(image);
}
```
