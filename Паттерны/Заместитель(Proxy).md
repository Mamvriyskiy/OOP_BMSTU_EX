>Заместитель (или Proxy) позволяет нам работать не с реальным объектом, а с другим объектом, который подменяет реальный. В каких целях это можно делать?

1. **Подменяющий объект может контролировать другой объект, задавать правила доступа к этому объекту**. Например, у нас есть разные категории пользователей. В зависимости от того, какая у пользователя категория категория, определять, давать доступ к самому объекту или не давать. _Это как защита_.
    
2. **Так как запросы проходят через заместителя, он может контролировать запросы, заниматься статистической обработкой**: считать количество запросов, какие запросы были и так далее.
    
3. **Разгрузка объекта с точки зрения запросов.** Дело в том, что реальные объекты какие-то операции могут выполнять крайне долго, например, обращение "поиск в базе чего-либо" или "обращение по сети куда-то". Это выполняется долго. **Proxy может сохранять предыдущий ответ и при следующем обращении смотреть, был ответ на этот запрос или не был**. Если ответ на этот вопрос был, он не обращается к самому хозяину, он заменяет его тем ответом, который был ранее. Естественно, если состояние объекта изменилось, Proxy должен сбросить ту историю, которую он накопил.

![[proxy.png]]

>Базовый класс Subject, реальный объект RealObject и объект Proxy, который содержит ссылку на объект, который он замещает. Когда мы работаем через указатель на базовый объект Subject, мы даже не можем понять, с кем мы реально работаем: с непосредственно объектом RealSubject или с его заместителем Proxy. А заместитель может выполнять те задачи, которые мы на него возложили.

>Если состояние RealObject изменилось, Прокси должен сбросить историю, которую он накопил.

*** 

**Преимущества:**
- Возможность контролировать какой-либо объект незаметно от клиента.
- Proxy может работать, когда объекта нет. Например, мы запросили объект, может быть объект был, но ответ для Proxy устарел, этого объекта нет. Proxy может вернуть нам этот ответ, указав, что он устаревший.
- Proxy может отвечать за жизненный цикл объекта. Он его может создавать и удалять.

**Недостатки:**
- Может увеличиться время доступа к объекту (так как идет дополнительная обработка через Proxy).
- Proxy должен хранить историю обращений, если такая задача стоит. Это влияет на память, она расходуется на хранение запросов и ответов на них.

```c++
# include <iostream>
# include <memory>
# include <map>
# include <random>

using namespace std;

class Subject
{
public:
	virtual ~Subject() = default;

	virtual pair<bool, double> request(size_t index) = 0;
	virtual bool changed() { return true; }
};

class RealSubject : public Subject
{
private:
	bool flag{ false };
	size_t counter{ 0 };

public:
	virtual pair<bool, double> request(size_t index) override;
	virtual bool changed() override;
};

class Proxy : public Subject
{
protected:
	shared_ptr<RealSubject> realsubject;

public:
	Proxy(shared_ptr<RealSubject> real) : realsubject(real) {}
};

class ConProxy : public Proxy
{
private:
	map<size_t, double> cache;

public:
	using Proxy::Proxy;

	virtual pair<bool, double> request(size_t index) override;
};

#pragma region Methods
bool RealSubject::changed()
{
	if (counter == 0)
	{
			flag = true;
	}
	if (++counter == 7)
	{
		counter = 0;
		flag = false;
	}
	return flag;
}


pair<bool, double> RealSubject::request(size_t index)
{
	random_device rd;
	mt19937 gen(rd());

	return pair<bool, double>(true, generate_canonical<double, 10>(gen));
}

pair<bool, double> ConProxy::request(size_t index)
{
	pair<bool, double> result;

	if (!realsubject)
	{
		cache.clear();

		result = pair<bool, double>(false, 0.);
	}
	else if (!realsubject->changed())
	{
		cache.clear();

		result = realsubject->request(index);

		cache.insert(map<size_t, double>::value_type(index, result.second));
	}
	else
	{
		map<size_t, double>::const_iterator it = cache.find(index);

		if (it != cache.end())
		{
			result = pair<bool, double>(true, it->second);
		}
		else
		{
			result = realsubject->request(index);

			cache.insert(map<size_t, double>::value_type(index, result.second));
		}
	}

	return result;
}
#pragma endregion

int main()
{
	shared_ptr<RealSubject> subject = make_shared<RealSubject>();
	shared_ptr<Subject> proxy = make_shared<ConProxy>(subject);
	
	for (size_t i = 0; i < 21; ++i)
	{
		cout << "( " << i + 1 << ", " << proxy->request(i % 3).second << " )" << endl;
	
		if ((i + 1) % 3 == 0)
			cout << endl;
	}
}
```