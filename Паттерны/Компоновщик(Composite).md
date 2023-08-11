### Введение

>Очень часто мы работаем со многими объектами однотипно, то есть выполняем над ними одни и те же операции. Более того, возникают ситуации, когда нужно выполнять над группой объектов эти действия совместно.

#### Пример

>Есть 3D модель, которую мы можем поворачивать, переносить. А если моделей несколько? У нас возникает необходимость объединять эти 3D модели в одну - сделать сборку и выполнять уже над этой сборкой совместные действия. У нас на сцене объектов может быть много, и естественно **хотелось бы рассматривать и каждый в отдельности объект, и совместно**.

#### Идея

>Вынести интерфейс, который предлагает контейнер (объект, включающий в себя другие объекты), на уровень базового класса

>Компоновщик компонует объекты в древовидную структуру, в которой над всей иерархией объектов можно выполнять такие же действия, как над простым элементом или над его частью.

#### Почему древовидная структура?

>Каждый композит может включать в себя как простые компоненты, так и другие композиты. Получается древовидная структура. Системе неважно, композит это или не композит. Она работает с любым элементом.

>Задача методов интерфейса для компонентов - пройтись по списку компонент и выполнить соответствующий метод.


![[compos.png]]

1. Базовый класс - Component. Нам должно быть безразлично, с каким объектом мы работаем: то ли это один компонент, то ли это объект, включающий в себя другие объекты (контейнер). Если это контейнер, то нам надо работать с содержимым контейнера, удалять, добавлять в него объекты. **Идея - вынести этот интерфейс на уровень базового класса** (добавление компонента - add(Component), удаление компонента - remove(Iterator), createIterator()). Нам надо четко понимать, когда мы работаем с каким-то компонентом, чем он является: один объект или контейнер. Для этого нам нужен метод isComposite(). То, что мы можем делать с Component - operation() - чисто виртуальные методы. Остальные (add, remove, и т. д.) мы будем реализовывать.
2. Leaf - простой компонент, его задачей будет только реализовать остальные методы - operation.
3. Composite - составной класс. Рализует все те методы, что есть в компоненте. Он содержит в себе список компонент.
***

**Преимущества:**
- Упрощает архитектуру клиента при работе со сложным деревом компонентов.
- Облегчает добавление новых компонентов.

**Недостатки:** 
- Никто не контролирует композит, нет ответственного за его создание.
- Если композит состоит из композитов и компонентов, то как отобрать только композиты?
- Сложно что-то удалять.
- Медленная работа.

```c++
# include <iostream>
# include <initializer_list>
# include <memory>
# include <vector>

using namespace std;

class Component;

using PtrComponent = shared_ptr<Component>;
using VectorComponent = vector<PtrComponent>;

class Component
{
public:
	using value_type = Component;
	using size_type = size_t;
	using iterator = VectorComponent::const_iterator;
	using const_iterator = VectorComponent::const_iterator;
	
	virtual ~Component() = default;
	
	virtual void operation() = 0;
	
	virtual bool isComposite() const { return false; }
	virtual bool add(initializer_list<PtrComponent> comp) { return false; }
	virtual bool remove(const iterator& it) { return false; }
	virtual iterator begin() const { return iterator(); }
	virtual iterator end() const { return iterator(); }
};

class Figure : public Component
{
public:
	virtual void operation() override { cout << "Figure method;" << endl; }
};

class Camera : public Component
{
public:
	virtual void operation() override { cout << "Camera method;" << endl; }
};

class Composite : public Component
{
private:
	VectorComponent vec;

public:
	Composite() = default;
	Composite(PtrComponent first, ...);

	void operation() override;
	
	bool isComposite() const override { return true; }
	bool add(initializer_list<PtrComponent> list) override;
	bool remove(const iterator& it) override { vec.erase(it); return true; }
	iterator begin() const override { return vec.begin(); }
	iterator end() const override { return vec.end(); }
};

# pragma region Methods
Composite::Composite(PtrComponent first, ...)
{
	for (shared_ptr<Component>* ptr = &first; *ptr; ++ptr)
	vec.push_back(*ptr);
}

void Composite::operation()
{
	cout << "Composite method:" << endl;
	for (auto elem : vec)
	elem->operation();
}

bool Composite::add(initializer_list<PtrComponent> list)
{
	for (auto elem : list)
	vec.push_back(elem);

	return true;
}

# pragma endregion

int main()
{
	using Default = shared_ptr<Component>;
	PtrComponent fig = make_shared<Figure>(), cam = make_shared<Camera>();
	auto composite1 = make_shared<Composite>(fig, cam, Default{});

	composite1->add({ make_shared<Figure>(), make_shared<Camera>() });
	composite1->operation();
	cout << endl;
	
	auto it = composite1->begin();
	
	composite1->remove(++it);
	composite1->operation();
	cout << endl;
	
	auto composite2 = make_shared<Composite>(make_shared<Figure>(), composite1, Default());
	
	composite2->operation();
}
```