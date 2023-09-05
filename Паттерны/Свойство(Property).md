>Паттерн Свойство (Property) представляет собой объединение двух методов – геттера и сеттера, что позволяет обеспечить сохранение целостности доступа к данным объекта.

**Преимущества паттерна:
- Позволяет инкапсулировать доступ к членам данных объекта
- Обеспечивает контроль доступа к данным для избегания ошибок при работе с объектами
- Позволяет добавить дополнительную логику при доступе к данным, например, проверку на корректность значений или логирования запроса доступа к данным.

**Недостатки паттерна:
- Приводит к увеличению количества типового кода, так как для каждого поля объекта требуется создание методов доступа
- Может усложнить понимание кода, так как доступ к данным может быть скрыт за методом

## Реализация простого свойства с возможностью записи и чтения

Свойство должно хранить в себе указатели на два члена класса – геттер и сеттер, а также перегружать оператор присваивания и приведения типа.

Простое свойство может быть использовано в том случае, если необходимо добавить некоторый функционал к установке и получению значения, относящегося к объекту класса.

## Свойства только для чтения, только для записи, для записи и чтения

Более продвинутым является разделение свойств на несколько видов в зависимости от возможности записывать или читать их значение.

Данные возможности могут быть использованы для реализации compute-only свойств, вычисляющих своё значение при вызове, либо для установки множества параметров объекта путем записи в одно свойство. Такие свойства могут быть реализованы при помощи специализации шаблона свойства различными тегами уровня доступа, либо при помощи метапрограммирования с удалением методов, которые не могут быть вызваны согласно установленному режиму.

```c++
# include <iostream>
# include <memory>

using namespace std;

template <typename Owner, typename Type>
class Property
{
	using Getter = Type(Owner::*)() const;
	using Setter = void (Owner::*)(const Type&);
	private:
	Owner* owner;
	Getter methodGet;
	Setter methodSet;

public:
	Property() = default;
	Property(Owner* const owr, Getter getmethod, Setter setmethod) : owner(owr), methodGet(getmethod), methodSet(setmethod) 
	{}

	void init(Owner* const owr, Getter getmethod, Setter setmethod)
	{
		owner = owr;
		methodGet = getmethod;
		methodSet = setmethod;
	}

	operator Type() { return (owner->*methodGet)(); }// Getter
	void operator=(const Type& data) { (owner->*methodSet)(data); }// Setter

	//Property(const Property&) = delete;
	//Property& operator=(const Property&) = delete;
};

class Object
{
private:
	double value;

public:
	Object(double v) : value(v) { Value.init(this, &Object::getValue, &Object::setValue); }

	double getValue() const { return value; }
	void setValue(const double& v) { value = v; }

	Property<Object, double> Value;
};

int main()
{
	Object obj(5.);
	
	cout << "value = " << obj.Value << endl;
	
	obj.Value = 10.;
	
	cout << "value = " << obj.Value << endl;
	
	unique_ptr<Object> ptr = make_unique<Object>(15.);
	
	cout << "value =" << ptr->Value << endl;
	
	obj = *ptr;
	obj.Value = ptr->Value;
}
```