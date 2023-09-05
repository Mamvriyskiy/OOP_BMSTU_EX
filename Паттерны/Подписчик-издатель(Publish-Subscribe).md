>Паттерн Подписчик-издатель (Publisher-subscriber) решает проблему реагирования на какие-либо события, либо изменений объекта. Это достигается за счёт того, что объект, с которым происходят изменения, сообщает об этом другим – тем, кто “подписан” на данные изменения.

>Группа объектов реагирует на один объект, на его изменения, издатель оповещает всех подписчиков, когда происхсодят изменения, вызывая их методы

![[ps.png]]

**Преимущества паттерна:**
- Издатели не зависят от подписчиков и всегда могут отправлять запросы.
- Можно динамически подписываться и отписываться от издателей.

**Недостатки паттерна:**
- Каждый издатель должен держать список своих подписчиков и методов, которые должны вызываться.
- Нет порядка в оповещении подписчиков.
- Если подписчики тоже могут быть издателями, возможны утечки памяти из-за удержания shared_ptr на подписчиков, либо бесконечные циклы при обработке события.

## Варианты реализации и случаи применения:

>В различных языках существуют разные реализации данного паттерна, отличающиеся одним важным моментом – зависит ли время жизни подписчика от времени жизни издателя. Это верно, например, в языке C# и библиотеке Qt. В C++ можно различными путями подойти к этому вопросу при помощи использования shared_ptr или weak_ptr, что зависит от решаемой задачи. В случае реализации с удержанием объекта, необходимо отписываться от события, когда оно более не необходимо.

- Удержание подписчиков по weak_ptr. Данный подход применяется, если источник.
- Удержание подписчиков по shared_ptr. Данный подход применим, например, при реализации подсистем программы, где мы обеспечиваем существование всех компонентов подсистемы, подписавшихся на события.

> Универсальный шаблонный класс издателя, поддерживающий работу с weak_ptr и shared_ptr.

```c++
# include <iostream>
# include <memory>
# include <vector>

using namespace std;

class Subscriber;

using Reseiver = Subscriber;

class Publisher
{
	using Action = void(Reseiver::*)();
	using Pair = pair<shared_ptr<Reseiver>, Action>;
private:
	vector<Pair> callback;

	int indexOf(shared_ptr<Reseiver> r);

public:
	bool subscribe(shared_ptr<Reseiver> r, Action a);
	bool unsubscribe(shared_ptr<Reseiver> r);
	void run();
};

class Subscriber
{
public:
	virtual ~Subscriber() = default;
	
	virtual void method() = 0;
};

class ConSubscriber : public Subscriber
{
public:
	void method() override { cout << "method;" << endl; }
};

# pragma region Methods Publisher
bool Publisher::subscribe(shared_ptr<Reseiver> r, Action a)
{
	if (indexOf(r) != -1) return false;
	
	Pair pr(r, a);
	
	callback.push_back(pr);
	
	return true;
}

bool Publisher::unsubscribe(shared_ptr<Reseiver> r)
{
	int pos = indexOf(r);
	
	if (pos != -1)
		callback.erase(callback.begin() + pos);
	
	return pos != -1;
}

void Publisher::run()
{
	cout << "Run:" << endl;
	for (auto elem : callback)
	((*elem.first).*(elem.second))();
}

int Publisher::indexOf(shared_ptr<Reseiver> r)
{
	int i = 0;
	for (auto it = callback.begin(); it != callback.end() && r != (*it).first; i++, ++it);
	
	return i < callback.size() ? i : -1;
}

# pragma endregion

int main()
{
	shared_ptr<Subscriber> subscriber1 = make_shared<ConSubscriber>();
	shared_ptr<Subscriber> subscriber2 = make_shared<ConSubscriber>();
	shared_ptr<Publisher> publisher = make_shared<Publisher>();
	
	publisher->subscribe(subscriber1, &Subscriber::method);
	
	if (publisher->subscribe(subscriber2, &Subscriber::method))
		publisher->unsubscribe(subscriber1);
	
	publisher->run();
}
```