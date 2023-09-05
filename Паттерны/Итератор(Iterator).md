>Паттерн Итератор (Iterator) позволяет вынести в отдельную сущность функционал получения доступа к элементам контейнерного класса.

**Преимущества паттерна:
- Упрощает работу с коллекциями, позволяя обходить их элементы без знания о внутренней структуре коллекции
- Позволяет работать с различными типами коллекций независимо от их реализации
- Позволяет реализовывать различные алгоритмы обработки коллекций

**Недостатки паттерна:
- Добавление новых типов коллекций может потребовать изменения кода итератора
- Итератор может быть стать ненадежным (инвалидация) в случае изменения коллекции во время обхода
- Неоднозначность определения “конечного” итератора

## Варианты реализации и случаи применения:

Паттерн поддерживается в стандартной библиотеке C++, так, предоставлены различные варианты создания своих итераторов на основе тегов, либо концептов (начиная с C++20). Помимо этого, присутствует синтаксический сахар в виде цикла for each.

```c++
for (auto elem : container)
{
	cout << elem;
}

for (Iterator <T> it = container.begin(); it != container.end(); ++it)
{
	auto elem it;
	cout << elem;
}
```

#### Пример реализации простого итератора для обертки над массивом

```c++
template <typename T>
class ArrayIterator;

template <typename T>
class Array {
	using iterator = ArrayIterator<T>;
	using const_iterator = ArrayIterator<const T>;
	using value_type = T;

private:
	std::unique_ptr<T[]> data;
	size_t size;

public:
	Array(size_t size) : data(std::make_unique<T[]>(size)), size(size) { }
	T& operator[](size_t index) { return data[index]; }
	const T& operator[](size_t index) const { return data[index]; }
	size_t getSize() const { return size; }
	
	ArrayIterator<T> begin() { return ArrayIterator<T>(*this, 0); }
	ArrayIterator<T> end() { return ArrayIterator<T>(*this, size); }

};

template <typename T>
class ArrayIterator {
	using iterator_category = std::forward_iterator_tag;
	using difference_type = std::ptrdiff_t;
	using value_type = T;
	using pointer = T*;
	using reference = T&;
	
private:
	Array<T>& array;
	size_t index;

public:
	ArrayIterator(Array<T>& array, size_t index) : array(array), index(index) { }
	bool operator!=(const ArrayIterator& other) const { return index != other.index; }
	ArrayIterator& operator++() { ++index; return *this; }

	T& operator*() { return array[index]; }
	const T& operator*() const { return array[index]; }
	T* operator->() { return &array[index]; }
	const T* operator->() const { return &array[index]; }
};
```