#### Причины использования

Проблема, с которой мы сталкиваемся - это когда нам надо создавать объекты, и эти объекты в разных местах программы мы используем по-разному. Грубо говоря, **один объект выступает в нескольких ролях**. Из этого следует:
- Для каждой роли используется свой интерфейс. В принципе, эти интерфейсы могут пересекаться. Но, если мы будем создавать такой класс, мы получим **избыточный интерфейс**.
- Как правило, эти роли завязаны еще под ответственность. **Один и тот же объект будет иметь несколько ответственностей**, что с точки зрения ООП очень плохо. Каждый объект должен иметь только одну ответственностей.
#### Идея

У класса был один интерфейс. Мы этот интерфейс подменяем другим интерфейсом, чтобы в зависимости от ситуации использовать этот объект по-разному.
### Использование

Адаптер надо применять в следующих ситуациях:
1. Один объект может выступать в нескольких ролях.
2. Нам нужно внедрить в систему сторонние классы, имеющие другой интерфейс.
3. Мы, используя полиморфизм, сформировали интерфейс для базового класса. Но, какие-то определенные сущности, определенные от базового класса, должны поддерживать еще какой то функционал. Мы не можем расширить этот функционал и изменять написанный код. Мы можем эту проблему решить добавлением функционала на основе того, что есть, за счет адаптера. Причем это можно сделать не для всей иерархии, а для определенного объекта определенного класса или группы классов.

>У нас есть наш объект, и мы хотим подменить интерфейс этого объекта. Мы будем работать с этим объектом через объект другого класса. Таким образом, **мы любой объект можем внедрить в любое место, поменяв его интерфейс.**

![[ad.png]]

>Базовый класс - AdapterA, задача которого - подменить интерфейс.
  Базовый класс, который нам надо адаптировать - Adapter, его интерфейс надо подменить.
  Класс, который решает эту проблему - ConAdapterA. Он будет подменять интерфейс.

**Преимущества:**
- Позволяет не возлагать на один объект несколько ролей
- Разнесение ответственностей по классам
- Использование сторонних разработок, нестандартных
- Отделяет и скрывает от клиента подробности различных интерфейсов
- Позволяет адаптировать интерфейс к требуемому
- Позволяет независимо развивать различные ответственности сущности
- Расширение интерфейса можно отнести к адаптеру

**Недостатки:**
- Нужно создавать много классов, что увеличивает время и память
- Один класс может поддерживать несколько интерфейсов, они могут пересекаться.  Дублирование кода.
- Часто адаптер должен иметь доступ к реализации класса.

```c++
# include <iostream>
# include <memory>

using namespace std;

class BaseAdaptee
{
public:
	virtual ~BaseAdaptee() = default;

	virtual void specificRequest() = 0;
};

class ConAdaptee : public BaseAdaptee
{
public:
	virtual void specificRequest() override { cout << "Method ConAdaptee;" << endl; }
};

class Adapter
{
public:
	virtual ~Adapter() = default;

	virtual void request() = 0;
};

class ConAdapter : public Adapter
{
private:
	shared_ptr<BaseAdaptee>  adaptee;

public:
	ConAdapter(shared_ptr<BaseAdaptee> ad) : adaptee(ad) {}

	virtual void request() override;
};

# pragma region Methods
void ConAdapter::request()
{
	cout << "Adapter: ";

	if (adaptee)
	{
		adaptee->specificRequest();
	}
	else
	{
		cout << "Empty!" << endl;
	}
}

# pragma endregion

int main()
{
	shared_ptr<BaseAdaptee> adaptee = make_shared<ConAdaptee>();
	shared_ptr<Adapter> adapter = make_shared<ConAdapter>(adaptee);

	adapter->request();
}

```
