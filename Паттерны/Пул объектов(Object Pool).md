>Пул объектов предоставляет набор готовых объектов, которые мы можем использовать
#### Пример использования

>Многопроцессорная система. Количество заданий, которое мы должны создать, должно быть не больше количества процессоров. Чтобы каждый процессор выполнял свою задачу.
#### Когда надо использовать?

>Когда создание или уничтожение какого-либо объекта - трудоемкий процесс и надо "держать" определенное количество объектов в системе.

>Если нам необходим, возможно ограниченный, набор определенных объектов. При запросе объекта мы даем его из этого набора, после использования возвращаем назад, при этом вернув в исходное состояние.

>Используется, когда создание или уничтожение какого-либо объекта - трудоемкий процесс и надо "держать" определенное количество объектов в системе.
#### Задачи:
- Он держит эти объекты.
- Может их создавать (то есть может расширяться).
- По запросу отдает объект.
- Если клиенту этот объект не нужен, он может его вернуть в пул.
- Исходя из пунктов 3 и 4, для каждого включенного объекта в пул мы должны установить, используется он или не используется. Клиент может принимать объект и возвращать его в пул объектов.

**Преимущества:**
- улучшает производительность, так как объекты не создаются и не уничтожаются

**Недостатки:**
- после использования объектов мы возвращаем его в пул, здесь возможна утечка информации. Объект также необходимо вернуть в исходное состояние.

![[op.png]]

>Пулл объектов - контейнерный класс, удобно использовать итераторы. Необходимо знать, занят объект или нет, используем пару: ключ (bool) и объект.

```c++
# include <iostream>
# include <memory>
# include <iterator>
# include <vector>

using namespace std;

template <typename T>
concept PoolObject = requires(T t)
{
	t.clear();
};

class Product
{
private:
	static size_t count;

public:
	Product() { cout << "Constructor(" << ++count << ");" << endl; }
	~Product() { cout << "Destructor(" << count-- << ");" << endl; }

	void clear() { cout << "Method clear: 0x" << this << endl; }
};

size_t Product::count = 0;

template <PoolObject Type>
class Pool
{
public:
	static shared_ptr<Pool<Type>> instance();

	shared_ptr<Type> getObject();
	bool releaseObject(shared_ptr<Type>& obj);
	size_t count() const { return pool.size(); }

	Pool(const Pool&) = delete;
	Pool& operator =(const Pool&) = delete;

private:
	vector<pair<bool, shared_ptr<Type>>> pool;

	Pool() {}

	pair<bool, shared_ptr<Type>> create();
	
	template <typename Type>
	friend ostream& operator << (ostream& os, const Pool<Type>& pl);
};

# pragma region ObjectPool class Methods 
template <PoolObject Type>
shared_ptr<Pool<Type>> Pool<Type>::instance()
{
	static shared_ptr<Pool<Type>> myInstance(new Pool<Type>());
	
	return myInstance;
}

template <PoolObject Type>
shared_ptr<Type> Pool<Type>::getObject()
{
	size_t i;
	for (i = 0; i < pool.size() && pool[i].first; ++i);
	
	if (i < pool.size())
	{
		pool[i].first = true;
	}
	else
	{
		pool.push_back(create());
	}
	
	return pool[i].second;
}

template <PoolObject Type>
bool Pool<Type>::releaseObject(shared_ptr<Type>& obj)
	{
	size_t i;
	for (i = 0; pool[i].second != obj && i < pool.size(); ++i);
	
	if (i == pool.size()) return false;
	
	obj.reset();
	pool[i].first = false;
	pool[i].second->clear();
	
	return true;
}

template <PoolObject Type>
pair<bool, shared_ptr<Type>> Pool<Type>::create()
{
	return { true, make_shared<Type>() };
}

# pragma endregion

template <typename Type>
ostream& operator << (ostream& os, const Pool<Type>& pl)
{
	for (auto elem : pl.pool)
	os << "{" << elem.first << ", 0x" << elem.second << "} ";

	return os;
}

int main()
{
	shared_ptr<Pool<Product>> pool = Pool<Product>::instance();
	
	vector<shared_ptr<Product>> vec(4);
	
	for (auto& elem : vec)
	elem = pool->getObject();
	
	pool->releaseObject(vec[1]);
	
	cout << *pool << endl;
	
	shared_ptr<Product> ptr = pool->getObject();
	vec[1] = pool->getObject();
	
	cout << *pool << endl;
}
```

