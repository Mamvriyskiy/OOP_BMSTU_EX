## Идея

>Идея паттерна Стратегия (Strategy) заключается в том, что мы оборачиваем функцию в класс и используем его в другом классе. Данный паттерн по сути представляет собой вырожденный паттерн Мост, в котором выносится не вся реализация, а лишь один метод.

>Нам во время выполнения надо менять реализацию какого-либо метода. Мы можем делать производные классы с разными реализациями и осуществлять "миграцию" между классами во врем выполнения - это неудобно, ибо мы начинаем работать с конкретными типами (классами).

>То, что может меняться (это действие) вынести в отдельный класс, который будет выполнять только это действие. Мы можем во время выполнения подменять одно на другое.

>Паттерн Стратегия определяет семейство всевозможных алгоритмов.

Преимущества:
- Избавляемся от разрастания иерархии классов (не приходится создавать практически одинаковые классы с одним разным методом)
- Избавляемся от дублирования кода(пересечение интерфейсов ролей)
- Можно использовать одну и ту же стратегию в не связанных друг с другом классах
- Имеем возможность легкого добавления или изменения функционала во время выполнения

Недостатки:
- Конкретная стратегия может не работать с данными определенного класса, что приводит к появлению зависимостей между конкретными сущностями и стратегиями.
- Необходимость установления дружественных связей, либо реализация передачи данных в стратегию.

![[str.png]]
## Варианты реализации и случаи применения:

#### Держим указатель на стратегию

В данном подходе объект держит указатель на стратегию. При вызове метода используется текущая установленная стратегия.

Данный подход применяется в том случае, если исходя из семантики программы требуется установить объекту определенное поведение, которое ему присуще.

```c++
class Strategy
{
public:
	virtual ~Strategy() = default;

	virtual void algorithm() = 0;
};

class ConStrategy1 : public Strategy
{
public:
	virtual void algorithm() override { cout << "Algorithm 1;" << endl; }

};

class ConStrategy2 : public Strategy
{
public:
	virtual void algorithm() override { cout << "Algorithm 2;" << endl; }
};  

class ContextHold
{
private:
	std::shared_ptr<Strategy> strategy;
public:
	explicit ContextHold(std::shared_ptr<Strategy> st) : strategy(std::move(st)) {}
	void algorithmStrategy() { strategy->algorithm(); }
};
```
#### Передаем стратегию при вызове

В данном подходе конкретная стратегия передаётся объекту при вызове соответствующего метода.

Данный подход может применяться в том случае, если имеется внешний класс, принимающий решение об используемой стратегии и в том случае, если нет необходимости сохранять её в классе для дальнейшего использования. 

```c++
class ContextPass
{
public:
	void algorithmStrategy(std::shared_ptr<Strategy>& strategy) { strategy->algorithm(); }
};
```
#### Статический полиморфизм

Статический полиморфизм может использоваться для обеспечения связывания на этапе компиляции в том случае, если мы знаем, какая стратегия будет вызываться, для чего можно воспользоваться шаблонным классом или методом.

Первый случай позволит жестко связать объект с используемой стратегий, в то время как второй позволяет выполнять связывание на месте вызова, таким образом, позволяя вызывать различные стратегии для одного объекта. Оба варианта реализации приведены на листинге 3.

```c++
template <typename TStrategy>
class ContextStaticA
{

private:
	std::unique_ptr<TStrategy> strategy;

public:
	ContextStaticA() : strategy(new TStrategy()) {}
	void algorithmStrategy() { strategy->algorithm(); }

};

class ContextStaticB
{

public:
	template<typename TStrategy>

	void algorithmStrategy() { TStrategy().algorithm(); };
	
	template<typename TStrategy>
	void algorithmStrategy(TStrategy &strategy) { strategy.algorithm(); }
};
```


Пример 1:
```c++
# include <iostream>
# include <memory>

using namespace std;

class Strategy
{
public:
	virtual ~Strategy() = default;

	virtual void algorithm() = 0;
};

class ConStrategy1 : public Strategy
{
public:
	void algorithm() override { cout << "Algorithm 1;" << endl; }
};

class ConStrategy2 : public Strategy
{
public:
	void algorithm() override { cout << "Algorithm 2;" << endl; }
};

class Context
{
protected:
	unique_ptr<Strategy> strategy;

public:
	explicit Context(unique_ptr<Strategy> ptr = make_unique<ConStrategy1>())
			: strategy(move(ptr)) {}
	virtual ~Context() = default;

	virtual void algorithmStrategy() = 0;
};

class Client1 : public Context
{
public:
	using Context::Context;

	void algorithmStrategy() override { strategy->algorithm(); }
};

int main()
{
	shared_ptr<Context> obj = make_shared<Client1>(make_unique<ConStrategy2>());
	
	obj->algorithmStrategy();
}

```

Пример 2:
```c++
# include <iostream>
# include <memory>
# include <vector>

using namespace std;

class Strategy
{
public:
	virtual ~Strategy() = default;

	virtual void algorithm() = 0;
};

class ConStrategy1 : public Strategy
{
public:
	void algorithm() override { cout << "Algorithm 1;" << endl; }
};

class ConStrategy2 : public Strategy
{
public:
	void algorithm() override { cout << "Algorithm 2;" << endl; }
};

class Context
{
public:
	virtual void algorithmStrategy(shared_ptr<Strategy> strategy) = 0;
};

class Client1 : public Context
{
public:
	void algorithmStrategy(shared_ptr<Strategy> strategy = make_shared<ConStrategy1>()) override
	{
	strategy->algorithm();
	}
};

int main()
{
	shared_ptr<Context> obj = make_shared<Client1>();
	shared_ptr<Strategy> strategy = make_shared<ConStrategy2>();
	
	obj->algorithmStrategy(strategy);
}
```