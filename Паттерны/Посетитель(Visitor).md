>Следующая проблема - связанная с изменением интерфейса объектов. Если мы используем полиморфзим, мы не можем в производном кассе ни сузить, ни расширить интерфейс, так как он должен четко поддерживать интерфейс базового класса. Если нам необходимо расширить интерфейс, можно использовать паттерн Визитёр. Он позволяет **во время выполнения** (в отличие от паттерна Адаптера, который решает эту проблему до выполнения) подменить или расширить функционал.

![[vis.png]]

>Визитёр один функционал собирает в одно место для разных классов. Для каждого такого класса/подкласса есть свой метод, который принимает элемент этого подкласса. Конкретный визитёр уже реализует этот функционал.

**Преимущества паттерна:**
- Объединение разных иерархий в одну (проблема стратегии)
- Значительное упрощение схемы использования
- Отсутствие оберточных функций (проблема адаптера)

**Недостатки паттерна:**
- Расширение иерархии, добавление новых классов приводит к необходимости модификации посетителей.
- Проблема связи на уровне базовых классов.
- При изменении иерархии посетитель не сработает.
- Необходимость установления дружественных связей для обеспечения доступа к реализации.
## Простая реализация посетителя

>В данном варианте используется базовый класс посетителя и метод сущности, принимающий его конкретную реализацию, приведённую к базе.

>Данный подход может быть использован для реализации единой логики при посещении множества объектов.

```c++
#include <iostream>
#include <memory>
#include <vector>

using namespace std;

// Предварительные объявления классов
class Circle;
class Rectangle;

// Базовый класс посетителя (Visitor)
class Visitor
{
public:
    virtual ~Visitor() = default;

    // Чисто виртуальные функции для посещения Circle и Rectangle
    virtual void visit(Circle& ref) = 0;
    virtual void visit(Rectangle& ref) = 0;
};

// Базовый класс фигуры (Shape)
class Shape
{
public:
    virtual ~Shape() = default;

    // Чисто виртуальная функция для принятия посетителя
    virtual void accept(shared_ptr<Visitor> visitor) = 0;
};

// Класс Circle, наследующийся от Shape
class Circle : public Shape
{
public:
    // Реализация функции accept для Circle
    void accept(shared_ptr<Visitor> visitor) override { visitor->visit(*this); }
};

// Класс Rectangle, наследующийся от Shape
class Rectangle : public Shape
{
public:
    // Реализация функции accept для Rectangle
    void accept(shared_ptr<Visitor> visitor) override { visitor->visit(*this); }
};

// Класс ConVisitor, наследующийся от Visitor
class ConVisitor : public Visitor
{
public:
    // Реализация функций visit для Circle и Rectangle
    void visit(Circle& ref) override { cout << "Circle;" << endl; }
    void visit(Rectangle& ref) override { cout << "Rectangle;" << endl; }
};

// Класс Figure, наследующийся от Shape
class Figure : public Shape
{
    using Shapes = vector<shared_ptr<Shape>>;

private:
    Shapes shapes;

public:
    // Конструктор, принимающий список shared_ptr<Shape>
    Figure(initializer_list<shared_ptr<Shape>> list)
    {
        for (auto&& elem : list)
            shapes.emplace_back(elem);
    }

    // Реализация функции accept для Figure
    void accept(shared_ptr<Visitor> visitor) override
    {
        for (auto& elem : shapes)
            elem->accept(visitor);
    }
};

int main()
{
    // Создание фигуры из кругов и прямоугольников
    shared_ptr<Shape> figure = make_shared<Figure>(
        initializer_list<shared_ptr<Shape>>(
            { make_shared<Circle>(), make_shared<Rectangle>(), make_shared<Circle>() }
        ));

    // Создание посетителя (ConVisitor)
    shared_ptr<Visitor> visitor = make_shared<ConVisitor>();

    // Применение посетителя к фигуре
    figure->accept(visitor);
}
```

## Посетитель с приведением типа между базовыми классами

>Данный подход использует пустой абстрактный класс посетителя, который дополняется посетителями конкретных типов при помощи реализации идиомы MixIn.

>Данный подход может быть использован, в случае, если требуется создавать посетители с сильно разнящимся поведением относительно посещаемых типов, когда мы не хотим размещать разную логику обработки сущностей рядом в одном классе.

```c++
# include <iostream>
# include <vector>
# include <memory>

using namespace std;

class AbstractVisitor
{
public:
    virtual ~AbstractVisitor() = default;
};

template <typename T>
class Visitor
{
public:
    virtual ~Visitor() = default;

    virtual void visit(const T&) const = 0;
};

class Shape
{
public:
    Shape() = default;
    virtual ~Shape() = default;

    virtual void accept(const AbstractVisitor&) const = 0;
};

class Circle : public Shape
{
private:
    double radius;

public:
    Circle(double radius) : radius(radius) {}

    void accept(const AbstractVisitor& v) const override
    {
        auto cv = dynamic_cast<const Visitor<Circle>*>(&v);

        if (cv)
        {
            cv->visit(*this);
        }
    }
};

class Square : public Shape
{
private:
    double side;

public:
    Square(double side) : side(side) {}

    void accept(const AbstractVisitor& v) const override
    {
        auto cv = dynamic_cast<const Visitor<Square>*>(&v);

        if (cv)
        {
            cv->visit(*this);
        }
    }
};

class DrawCircle : public Visitor<Circle>
{
    void visit(const Circle& circle) const override
    {
        cout << "Circle" << endl;
    }
};

class DrawSquare : public Visitor<Square>
{
    void visit(const Square& circle) const override
    {
        cout << "Square" << endl;
    }
};

class Figure : public Shape
{
    using Shapes = vector<shared_ptr<Shape>>;

private:
    Shapes shapes;

public:
    Figure(initializer_list<shared_ptr<Shape>> list)
    {
        for (auto&& elem : list)
            shapes.emplace_back(elem);
    }

    void accept(const AbstractVisitor& visitor) const override
    {
        for (auto& elem : shapes)
            elem->accept(visitor);
    }
};

class Draw : public AbstractVisitor, public DrawCircle, public DrawSquare {};

int main()
{
    shared_ptr<Shape> figure = make_shared<Figure>(
        initializer_list<shared_ptr<Shape>>({ make_shared<Circle>(1), make_shared<Square>(2) })
    );

    figure->accept(Draw{});
}

```

## Шаблонный посетитель с использованием CRTP

>Данная реализация прибегает к вариативным шаблонам классов для определения типа посетителя, который посещает заданные классы. За счёт использования CRTP можно избавиться от реализации метода accept в каждой сущности, перенеся ответственность за это на класс Visitable.

```c++
# include <iostream>
# include <memory>
# include <initializer_list>
# include <vector>

using namespace std;

template <typename... Types>
class Visitor;

template <typename Type>
class Visitor<Type>
{
public:
	virtual void visit(Type& t) = 0;
};

template <typename Type, typename... Types>
class Visitor<Type, Types...> : public Visitor<Types...>
{
public:
	using Visitor<Types...>::visit;
	virtual void visit(Type& t) = 0;
};

using ShapeVisitor = Visitor<class Figure, class Camera>;

class Point {};

class Shape
{
public:
	Shape(const Point& pnt) : point(pnt) {}
	virtual ~Shape() = default;

	const Point& getPoint() const { return point; }
	void setPoint(const Point& pnt) { point = pnt; }

	virtual void accept(shared_ptr<ShapeVisitor> v) = 0;

private:
	Point point;
};

template <typename Derived>
class Visitable : public Shape
{
public:
	using Shape::Shape;

	void accept(shared_ptr<ShapeVisitor> v) override
	{
		v->visit(*static_cast<Derived*>(this));
	}
};

class Figure : public Visitable<Figure>
{
	using Visitable<Figure>::Visitable;
};

class Camera : public Visitable<Camera>
{
	using Visitable<Camera>::Visitable;
};

class Composite : public Shape
{
	using Shapes = vector<shared_ptr<Shape>>;

private:
	Shapes shapes{};

public:
	Composite(initializer_list<shared_ptr<Shape>> list) : Shape(Point{})
	{
		for (auto&& elem : list)
		shapes.emplace_back(elem);
	}

	void accept(shared_ptr<ShapeVisitor> visitor)  override
	{
		for (auto& elem : shapes)
		elem->accept(visitor);
	}
};

class DrawVisitor : public ShapeVisitor
{
public:
	void visit(Figure& fig) override { cout << "Draws a figure;" << endl; }
	void visit(Camera& fig) override { cout << "Draws a camera;" << endl; }
};

int main()
{
	Point p;
	shared_ptr<Composite> figure = make_shared<Composite>(
	initializer_list<shared_ptr<Shape>>(
	{ make_shared<Figure>(p), make_shared<Camera>(p), make_shared<Figure>(p) }
	));

	shared_ptr<ShapeVisitor> visitor = make_shared<DrawVisitor>();

	figure->accept(visitor);
}

```