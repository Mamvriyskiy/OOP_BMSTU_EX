>В случае, если мы желаем принимать решение об использовании в одном месте, а само действие – в другом, возникает понятие запроса.

>Паттерн Команда (Command) решает проблему исполнения запроса в случае, если мы четко знаем, что надо сделать и какой объект это может выполнить.

>Паттерн имеет один основной метод – execute. Паттерн несёт указатель, либо другой идентификатор объекта, а также указывает на то, какое действие с какими параметрами должно быть выполнено с объектом.

![[command.png]]

>Запрос, или команда, идет в виде объекта. Базовый класс - и у нас может быть не одна команда, а несколько, в зависимости от того, что нам нужно. Команда может нести данные, может нести _что надо сделать_, (указатель на метод какого-либо объекта, например.)

**Преимущества паттерна:**
- Унификация обработки событий или запросов к системе.
- Уменьшается связанность между классами.
- Возможно управление выполнением команды, например,о постановка команд в очередь с приоритетом.
- Возможность комбинирования с другими паттернами, например с композитом – возможно формирование сложных команд, состоящих из других команд.

**Недостатки паттерна:**
- Требуется изначально знать, кто может обработать команду и какое конкретное действие должно быть совершено.
- Проблема возникает в случае, если необходимо, чтобы команда была обработана несколькими объектами.
## Варианты реализации и случаи применения:

#### Известен объект, которому предназначена команда

>Данный подход требует хранения объекта, над которым будет выполнена команда в самой команде, предоставляя, по сути, отложенный вызов некоторого действия.

>Может использоваться тогда, когда мы заранее уверены в том, что потребуется некоторое действие и ожидаем указания выполнить команду.

```c++
# include <iostream>
# include <memory>
# include <vector>
# include <initializer_list>

using namespace std;

class Command
{
public:
	virtual ~Command() = default;

	virtual void execute() = 0;
};

template <typename Reseiver>
class SimpleCommand : public Command
{
	using Action = void(Reseiver::*)();
	using Pair = pair<shared_ptr<Reseiver>, Action>;

private:
	Pair call;

public:
	SimpleCommand(shared_ptr<Reseiver> r, Action a) : call(r, a) {}

	void execute() override { ((*call.first).*call.second)(); }
};

class CompoundCommand : public Command
{
	using VectorCommand = vector<shared_ptr<Command>>;

private:
	VectorCommand vec;

public:
	CompoundCommand(initializer_list<shared_ptr<Command>> lt);

	virtual void execute() override;
};

# pragma region Methods
CompoundCommand::CompoundCommand(initializer_list<shared_ptr<Command>> lt)
{
	for (auto&& elem : lt)
	vec.push_back(elem);
}

void CompoundCommand::execute()
{
	for (auto com : vec)
	com->execute();
}

# pragma endregion

class Object
{
public:
	void run() { cout << "Run method;" << endl; }
};

int main()
{
	shared_ptr<Object> obj = make_shared<Object>();
	shared_ptr<Command> command = make_shared<SimpleCommand<Object>>(obj, &Object::run);

	command->execute();
	
	shared_ptr<Command> complex(new CompoundCommand
	{
	make_shared<SimpleCommand<Object>>(obj, &Object::run),
	make_shared<SimpleCommand<Object>>(obj, &Object::run)
	});
	
	complex->execute();
}

```
#### Неизвестен объект, которому предназначена команда

>Данный подход подразумевает передачу объекта в метод execute во время выполнения команды.

>Сценарием использования является ситуация, при которой мы заранее знаем о требуемом действии, но ещё не определили, над каким объектом оно должно быть выполнено. 

```c++
# include <iostream>
# include <memory>

using namespace std;

template <typename Reseiver>
class Command
{
public:
	virtual ~Command() = default;
	virtual void execute(shared_ptr<Reseiver>) = 0;
};

template <typename Reseiver>
class SimpleCommand : public Command<Reseiver>
{
	using Action = void(Reseiver::*)();
	private:
	Action act;

public:
	SimpleCommand(Action a) : act(a) {}

	virtual void execute(shared_ptr<Reseiver> r) override { ((*r).*act)(); }
};

class Object
{
public:
	virtual void run() = 0;
};

class ConObject : public Object
{
public:
	void run() override { cout << "Run method;" << endl; }
};

int main()
{
	shared_ptr<Command<Object>> command = make_shared<SimpleCommand<Object>>(&Object::run);
	
	shared_ptr<Object> obj = make_shared<ConObject>();
	
	command->execute(obj);
}

```